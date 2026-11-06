---
title: "MACD初选与竞价9:20异动"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1h1525ci3mnkc/mindoc-1hp49sodmb7r8.html
fetched_at: 2026-11-06T06:03:45Z
---

# # MACD初选与竞价9:20异动

**使用TdxAiData接口**

一句话：公式只取最近一个已收盘交易日的选股结果，把 5000+ 只 A 股压到几百只，再只对这几百只取 9:15–9:25 竞价曲线，用 9:20 窗口判断异动。

1、确认选股范围

get_stock_list('5')

全部A股 5580 只

2、确定目标日

最近一个已收盘交易日

formula_by_src

按Time取目标日信号

3、监控竞价范围

stock_list

get_call_auction_batch

9:15-9:25 逐点曲线

9:20 窗口异动判定

4、监控结果输出

结果股票 + CSV

整条链路只依赖 tdxaidata 的 tqs，不需要启动通达信客户端。公式、竞价、快照三个接口都能批量，，竞价只对初选结果取数据。

### # 接口流动过程

| 环节 | 接口 | 输入 | 输出 | 2026-11-03 09:59 实测 |
|---|---|---|---|---|
| A 股池 | `get_stock_list("5")` | - | 代码列表 | 5580 只 |
| 目标日 | `get_trading_dates("SH", ...)` | 最近 40 天 | `YYYYMMDD` | 20261102 |
| 公式初选 | `formula_by_src(..., count=250)` | 5580 只，切 500 一片 | 目标日命中 | 约 0.6 秒，282 只 |
| 竞价 | `get_call_auction_batch(...)` | 282 只，切 300 一片 | 逐点曲线 | 282 只全部返回 |
| 9:20 判定 | 本地计算 | 曲线 + 昨收 | 结果表 | 40 只异动 |

### # 1. 第一步：取全部 A 股

```
pool = tqs.get_stock_list(market="5", list_type=0)
codes = [c for c in pool if c[-3:] in (".SH", ".SZ", ".BJ")]
market="5" 就是全部 A 股，返回的代码自带市场后缀。
```


2

3

不传 market 或传错分类，会拿到非股票品种；固定用 "5"。


返回里偶有非标准代码，按 .SH / .SZ / .BJ 后缀过滤一遍最稳。

代码必须带后缀，后续 formula_by_src、get_call_auction_batch 都依赖它。

### # 2. 第二步：公式初选（截止最近一个交易日）

2.1 先确定目标日

```
from datetime import datetime, time as dtime, timedelta
def last_completed_trading_day() -> str:
now = datetime.now()
start = (now - timedelta(days=40)).strftime("%Y%m%d")
end = now.strftime("%Y%m%d")
dates = tqs.get_trading_dates("SH", start, end) or []
if not dates:
return ""
if dates[-1] == end and now.time() < dtime(15, 0):
dates = dates[:-1]
return dates[-1]
get_trading_dates 返回 YYYYMMDD 字符串列表，包含当天。规则如下：
```


2

3

4

5

6

7

8

9

10

11

12

| 运行时间 | 目标日 |
|---|---|
| 交易日 15:00 前（含竞价、盘中、午休） | 上一个交易日 |
| 交易日 15:00 后 | 当天 |
| 非交易日 | 最近一个交易日 |

为什么是 15:00


A 股 15:00 收盘，日线才算走完。9:20 跑策略时当天日线还没形成，目标日必须是上一交易日。

2.2 再把 := 改掉

条件选股公式写法：

DIFF := EMA(CLOSE,12) - EMA(CLOSE,26);

DEA := EMA(DIFF,9);

CROSS(DIFF,DEA);

formula_by_src 是临时编译，不支持 :=，会直接报"在变量前又错误的定义一次变量"。必须把中间量写成输出线：

DIFF:EMA(CLOSE,12)-EMA(CLOSE,26);

DEA:EMA(DIFF,9);

MACD:CROSS(DIFF,DEA);

输出线之间可以互相引用，所以 DEA 能直接引用 DIFF，这一点已实测通过。

2.3 调用公式

```
res = tqs.formula_by_src(
formula_src=FORMULA_SRC,
stock_list=pool[0:500],
stock_period="1d",
count=250, # 关键：历史必须足够长
return_count=5, # 返回最近 5 根，用于按目标日取信号
return_date=True, # 必须带上 Time
formula_type=1, # 1 = 条件选股，只返回命中股票
domain=0,
)
```


2

3

4

5

6

7

8

9

10

count 不能太小


EMA 是递归指标，count 决定喂给公式多少历史。

2.4 按目标日取信号

formula_type=1 会返回最近 return_count 根里“曾经命中过”的股票，并带上完整 Time 序列。所以还要再按目标日过滤一次：

```
def pick_as_of(res: dict, target: str, signal_line: str = "MACD") -> list[str]:
codes = []
for code, cols in res.items():
if code in ("ErrorId", "Error") or not isinstance(cols, dict):
continue
times = [str(t) for t in cols.get("Time", [])]
if target not in times:
continue
index = times.index(target)
values = cols.get(signal_line, [])
if index < len(values) and float(values[index]) != 0:
codes.append(code)
return codes
```


2

3

4

5

6

7

8

9

10

11

12

13

SIGNAL_LINE 必须等于公式最后一条输出线的名字。本例是 MACD，它既是条件线，也是 0/1 信号。

选股模式会取整

formula_type=1 返回的 DIFF / DEA 是取整后的值（如 -26、-24），不能拿来当因子数值；但条件线 MACD 的 0/1 是准确的。要原始数值就用 formula_type=0。

- 第三步：得到MACD选股后的stock_list

stock_list = pick_as_of(res, target, signal_line="MACD")

到这里，5580 只已经被压到几百只。后面的竞价接口只对这些股票调用。

### # 4. 第四步：批量取竞价曲线

```
auction = tqs.get_call_auction_batch(
stock_list=codes,
field_list=["Time", "Price", "Volume", "LeaveQty"],
type=1, # 1 = 仅早盘竞价
date="", # 空 = 当天；1.3.0+ 可传历史日期
return_df=False, # 返回 {代码: {字段: 列表}}
)
```


2

3

4

5

6

7

| 字段 | 含义 | 使用注意 |
|---|---|---|
`Time` | `HHMMSS` 字符串 | 采样间隔 3/6/9/12/15 秒不等，不是严格 5 秒 |
`Price` | 虚拟匹配价 | 字符串，先转 `float` |
`Volume` | 虚拟匹配量 | 观察值是累计递增的，不是单点增量 |
`LeaveQty` | 未匹配量 | 可为负，负值表示卖方剩余 |

9:15–9:20 可撤单，假动作多；9:20–9:25 不可撤单，9:20 是第一个“相对真实”的锚点。


没有竞价数据的票可能返回空字典或缺失，后续按“无数据”跳过。

昨收不在竞价接口里，需要另取 get_market_snapshot_batch 的 LastClose。

### # 5. 第五步：9:20 异动判定

5.1 窗口定义

| 窗口 | 时间 | 用途 |
|---|---|---|
| 早盘基准 | 09:16:00–09:19:30 | 取中位数作为基准价，避免单点噪声 |
| 9:20 锚点 | 09:19:30–09:20:30 | 取窗口最后一个点作为 9:20 价 |
| 收盘竞价 | 09:20:30–09:25:00 | 取最后一个点作为竞价末价 |

5.2 四个指标

设 9:20 价为 𝑃920，早盘基准中位数为 𝑃𝑝𝑟𝑒，竞价末价为 𝑃𝑒𝑛𝑑，昨收为 𝑃𝑐𝑙𝑜𝑠𝑒：

竞价涨幅：𝑔𝑎𝑝 =(𝑃920/𝑃𝑐𝑙𝑜𝑠𝑒 −1) ×100

9:20 跳变：𝑗𝑢𝑚𝑝 =(𝑃920/𝑃𝑝𝑟𝑒 −1) ×100

20 分后上提：𝑝𝑜𝑠𝑡 =(𝑃𝑒𝑛𝑑/𝑃920 −1) ×100

未匹配比：𝑙𝑒𝑎𝑣𝑒 =|𝐿𝑒𝑎𝑣𝑒𝑄𝑡𝑦920|/𝑉𝑜𝑙𝑢𝑚𝑒920

5.3 判定规则

命中任意一条即视为异动：

高开且继续上提：𝑔𝑎𝑝 ≥3% 且 𝑝𝑜𝑠𝑡 ≥0.5%

9:20 单点跳变：|𝑗𝑢𝑚𝑝| ≥1.5%（ONLY_UP=True 时只看正向）

再加两个清洗条件：

竞价有效点至少 5 个；

9:20 累计匹配量 𝑉𝑜𝑙𝑢𝑚𝑒920 ≥20，过滤冷门票的虚拟撮合噪声。

5.4 为什么用窗口中位数

实测踩到的真实例子：920914.BJ 在 9:17–9:18 之间出现 14.99 → 10.43 → 14.99 的跳点，匹配量只有 2。若直接拿“9:20 前一个点”做基准，会算出 43.72% 的假跳变，排到结果第一。

换成“早盘窗口中位数 + 最小匹配量”后，这类噪声被过滤。北交所竞价数据整体更稀疏，实盘建议单独设阈值或直接剔除 .BJ。

9:20 是分水岭：之前可撤单，之后不可撤单，因此以 9:20 为锚点比 9:15 更有意义。


单点价格不可信，用窗口统计量；成交量是累计值，别当增量用。

“异动”不等于“买入信号”，输出表保留原始指标，方便你按自己的口径二次筛选。9.20之后的向上跳变和向下跳变都会选出来

### # 6. 第六步：输出

结果按评分降序落盘为 CSV，编码 utf-8-sig，Excel 直接打开不乱码。

| 列 | 含义 |
|---|---|
`代码` | 带市场后缀 |
`昨收` | 快照 `LastClose` |
`9:20价` | 9:20 锚点价 |
`9:20涨幅%` | 相对昨收 |
`跳变%` | 相对早盘窗口中位数 |
`9:20->9:25%` | 9:20 到竞价末价 |
`未匹配比` | 未匹配量 / 累计匹配量 |
`末价` | 竞价最后一点价格 |
`点数` | 有效竞价点数 |
`评分` | $gap + 2 \times post + jump - 2 \times leave$，仅用于排序 |

### # 7. 完整代码

```
# -*- coding: utf-8 -*-
"""A股 -> 公式初选 -> stock_list -> 竞价(9:20)异动 （tqs 全流程）
流程：
1. get_stock_list("5") 取全部 A 股
2. formula_by_src(..., formula_type=1) 跑公式，按最近一个已收盘交易日取信号
3. pick_as_of(...) 得到初选 stock_list
4. get_call_auction_batch(...) 批量取竞价逐点曲线
5. 9:20 窗口异动判定 输出结果股票
"""
from __future__ import annotations
import csv
import statistics
import sys
import time
from datetime import datetime, time as dtime, timedelta
from pathlib import Path
from typing import Any
from tdxaidata import tqs
sys.stdout.reconfigure(encoding="utf-8")
# ============================== 参数 ==============================
# 公式初选：MACD 金叉。注意 formula_by_src 不支持 :=，中间量必须写成输出线。
FORMULA_SRC = """DIFF:EMA(CLOSE,12)-EMA(CLOSE,26);
DEA:EMA(DIFF,9);
MACD:CROSS(DIFF,DEA);"""
FORMULA_TYPE = 1 # 1 = 条件选股，只返回命中股票
PERIOD = "1d"
LOOKBACK = 250 # 必须足够大：EMA 递归需要历史，count 太小会算错金叉
RETURN_BARS = 5 # 返回最近 5 根，用于按交易日取信号
SIGNAL_LINE = "MACD" # 公式最后一条输出线（条件线）
DIVIDEND_TYPE = 0
TARGET_DATE = "" # 空 = 自动取最近一个已收盘交易日；也可手动写 20261102
# 股票池：pool = 显式取 A 股再切 500 一片；domain = 服务端按域直扫（单次无 500 上限）
FORMULA_MODE = "pool" # "pool" 或 "domain"
DOMAIN = 6 # 6 = 沪深京 A 股，仅 FORMULA_MODE="domain" 时生效
# 批量接口分片
FORMULA_CHUNK = 500
AUCTION_CHUNK = 300
SNAPSHOT_CHUNK = 500
# 9:20 异动阈值
GAP_MIN = 3.0 # 9:20 价格相对昨收涨幅 >= 3%
POST_MIN = 0.5 # 9:20 -> 9:25 继续上移 >= 0.5%
JUMP_MIN = 1.5 # 9:20 相对早盘基准的跳变绝对值 >= 1.5%
ONLY_UP = False # True = 只保留向上跳变
MIN_POINTS = 5 # 竞价至少 5 个有效点
MIN_AUC_VOL = 20.0 # 9:20 累计匹配量下限，过滤冷门票噪声
# 时间窗口（HHMMSS）
PRE_START, PRE_END = 91600, 91930 # 早盘基准窗口
ANCHOR_START, ANCHOR_END = 91930, 92030 # 9:20 锚点窗口
AUCTION_START, AUCTION_END = 91500, 92500
OUT_DIR = Path(__file__).resolve().parent / "auction_out"
# =================================================================
def to_float(value: Any, default: float = 0.0) -> float:
try:
return float(value)
except (TypeError, ValueError):
return default
def to_time(value: Any) -> int:
digits = "".join(ch for ch in str(value) if ch.isdigit())
return int(digits.zfill(6)[-6:]) if digits else -1
def as_list(value: Any) -> list:
if value is None:
return []
if isinstance(value, (list, tuple)):
return list(value)
return [value]
def last_completed_trading_day() -> str:
now = datetime.now()
start = (now - timedelta(days=40)).strftime("%Y%m%d")
end = now.strftime("%Y%m%d")
dates = tqs.get_trading_dates("SH", start, end) or []
if not dates:
return ""
if dates[-1] == end and now.time() < dtime(15, 0):
dates = dates[:-1]
return dates[-1] if dates else ""
def resolve_target(result: dict, target: str) -> str:
dates = {
str(x)
for cols in result.values() if isinstance(cols, dict)
for x in cols.get("Time", [])
}
if target in dates:
return target
prior = [d for d in dates if d <= target]
if not prior:
return target
fallback = max(prior)
print(f"[公式] {target} 无数据，回退到 {fallback}")
return fallback
def step1_a_pool() -> list[str]:
pool = tqs.get_stock_list(market="5", list_type=0) or []
return [c for c in pool if isinstance(c, str) and c[-3:] in (".SH", ".SZ", ".BJ")]
def step2_formula(pool: list[str], target: str) -> list[str]:
started = time.time()
result: dict = {}
if FORMULA_MODE == "domain":
part = tqs.formula_by_src(
formula_src=FORMULA_SRC,
stock_list=[],
stock_period=PERIOD,
count=LOOKBACK,
return_count=RETURN_BARS,
return_date=True,
dividend_type=DIVIDEND_TYPE,
formula_type=FORMULA_TYPE,
domain=DOMAIN,
)
if isinstance(part, dict):
result = part
else:
for start in range(0, len(pool), FORMULA_CHUNK):
part = tqs.formula_by_src(
formula_src=FORMULA_SRC,
stock_list=pool[start:start + FORMULA_CHUNK],
stock_period=PERIOD,
count=LOOKBACK,
return_count=RETURN_BARS,
return_date=True,
dividend_type=DIVIDEND_TYPE,
formula_type=FORMULA_TYPE,
domain=0,
)
if isinstance(part, dict):
result.update(part)
if result.get("Error"):
print("[公式] Error:", str(result["Error"])[:300])
target = resolve_target(result, target)
codes = []
for code, cols in result.items():
if code in ("ErrorId", "Error") or not isinstance(cols, dict):
continue
times = [str(x) for x in cols.get("Time", [])]
if target not in times:
continue
index = times.index(target)
values = cols.get(SIGNAL_LINE, [])
if index < len(values) and to_float(values[index]) != 0:
codes.append(code)
print(f"[公式] 截止 {target} 命中 {len(codes)} 只，耗时 {time.time() - started:.2f}s")
return codes
def step4_auction(codes: list[str]) -> dict:
records: dict = {}
for start in range(0, len(codes), AUCTION_CHUNK):
part = tqs.get_call_auction_batch(
stock_list=codes[start:start + AUCTION_CHUNK],
field_list=["Time", "Price", "Volume", "LeaveQty"],
type=1,
date="",
return_df=False,
)
if isinstance(part, dict):
records.update({k: v for k, v in part.items() if isinstance(v, dict)})
print(f"[竞价] 返回 {len(records)} 只")
return records
def step4_last_close(codes: list[str]) -> dict:
closes: dict = {}
for start in range(0, len(codes), SNAPSHOT_CHUNK):
part = tqs.get_market_snapshot_batch(
codes[start:start + SNAPSHOT_CHUNK], ["LastClose"], return_df=False
)
if isinstance(part, dict):
closes.update({
k: to_float(v.get("LastClose"))
for k, v in part.items() if isinstance(v, dict)
})
return closes
def step5_analyze(code: str, raw: dict, last_close: float) -> dict | None:
times = [to_time(t) for t in as_list(raw.get("Time"))]
prices = [to_float(p) for p in as_list(raw.get("Price"))]
volumes = [to_float(v) for v in as_list(raw.get("Volume"))]
leaves = [to_float(x) for x in as_list(raw.get("LeaveQty"))]
points = [
(t, p, v, l)
for t, p, v, l in zip(times, prices, volumes, leaves)
if AUCTION_START <= t <= AUCTION_END and p > 0
]
if len(points) < MIN_POINTS:
return None
pre_prices = [p for t, p, _v, _l in points if PRE_START <= t < PRE_END]
anchor_points = [(p, v, l) for t, p, v, l in points if ANCHOR_START <= t <= ANCHOR_END]
if not pre_prices or not anchor_points:
return None
base_price = statistics.median(pre_prices)
price_920, vol_920, leave_920 = anchor_points[-1]
end_price = points[-1][1]
if vol_920 < MIN_AUC_VOL:
return None
gap_pct = (price_920 / last_close - 1) * 100 if last_close > 0 else 0.0
jump_pct = (price_920 / base_price - 1) * 100 if base_price > 0 else 0.0
post_pct = (end_price / price_920 - 1) * 100 if price_920 > 0 else 0.0
leave_ratio = abs(leave_920) / vol_920 if vol_920 > 0 else 0.0
jump_hit = jump_pct >= JUMP_MIN if ONLY_UP else abs(jump_pct) >= JUMP_MIN
is_anomaly = (gap_pct >= GAP_MIN and post_pct >= POST_MIN) or jump_hit
if not is_anomaly:
return None
score = gap_pct + post_pct * 2 + jump_pct - leave_ratio * 2
return {
"代码": code,
"昨收": round(last_close, 3),
"9:20价": round(price_920, 3),
"9:20涨幅%": round(gap_pct, 2),
"跳变%": round(jump_pct, 2),
"9:20->9:25%": round(post_pct, 2),
"未匹配比": round(leave_ratio, 2),
"末价": round(end_price, 3),
"点数": len(points),
"评分": round(score, 2),
}
def main() -> None:
pool = step1_a_pool()
print(f"[A股] {len(pool)} 只，公式模式={FORMULA_MODE}")
target = TARGET_DATE or last_completed_trading_day()
if not target:
print("取不到交易日历，流程结束")
return
print(f"[目标日] 最近一个已收盘交易日 = {target}")
codes = step2_formula(pool, target)
print(f"[stock_list] {len(codes)} 只")
if not codes:
print("无命中股票，流程结束")
return
auction = step4_auction(codes)
closes = step4_last_close(list(auction))
if not auction:
print("竞价接口无数据，流程结束")
return
rows = [
row
for code, raw in auction.items()
if (row := step5_analyze(code, raw, closes.get(code, 0.0)))
]
rows.sort(key=lambda item: item["评分"], reverse=True)
print(f"[结果] 9:20 异动 {len(rows)} 只")
OUT_DIR.mkdir(parents=True, exist_ok=True)
output = OUT_DIR / f"auction_920_{datetime.now():%Y%m%d_%H%M%S}.csv"
if rows:
with output.open("w", newline="", encoding="utf-8-sig") as handle:
writer = csv.DictWriter(handle, fieldnames=list(rows[0]))
writer.writeheader()
writer.writerows(rows)
print(f"[落盘] {output}")
for row in rows[:20]:
print(row)
if __name__ == "__main__":
main()
```


2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

90

91

92

93

94

95

96

97

98

99

100

101

102

103

104

105

106

107

108

109

110

111

112

113

114

115

116

117

118

119

120

121

122

123

124

125

126

127

128

129

130

131

132

133

134

135

136

137

138

139

140

141

142

143

144

145

146

147

148

149

150

151

152

153

154

155

156

157

158

159

160

161

162

163

164

165

166

167

168

169

170

171

172

173

174

175

176

177

178

179

180

181

182

183

184

185

186

187

188

189

190

191

192

193

194

195

196

197

198

199

200

201

202

203

204

205

206

207

208

209

210

211

212

213

214

215

216

217

218

219

220

221

222

223

224

225

226

227

228

229

230

231

232

233

234

235

236

237

238

239

240

241

242

243

244

245

246

247

248

249

250

251

252

253

254

255

256

257

258

259

260

261

262

263

264

265

266

267

268

269

270

271

272

273

274

275

276

277

278

279

280

281

282

283

284

285

286

287

288

289

290

291

292

293

294

295

296

### # 8. 参数怎么调

| 参数 | 默认 | 说明 |
|---|---|---|
`LOOKBACK` | 250 | 喂给公式的历史根数；MACD/KDJ 等递归指标不要低于 250 |
`RETURN_BARS` | 5 | 返回最近几根；目标日必须落在窗口内 |
`SIGNAL_LINE` | `"MACD"` | 公式最后一条输出线（条件线）的名字 |
`TARGET_DATE` | `""` | 空 = 自动；手动指定时必须落在最近 `RETURN_BARS` 根内 |
`GAP_MIN` | 3.0 | 调大只保留更高开；调小纳入平开异动 |
`POST_MIN` | 0.5 | 调大要求 9:20 后更强上提 |
`JUMP_MIN` | 1.5 | 调大只保留更剧烈的 9:20 跳变 |
`MIN_AUC_VOL` | 20 | 调大过滤更多冷门票；调小保留低流动性小票 |
`ONLY_UP` | `False` | 改 `True` 只看向上跳变 |
`FORMULA_MODE` | `"pool"` | 改 `"domain"` 可一次扫全市场 |

[!tip] 公式换别的也一样


把`FORMULA_SRC`

换成任意条件选股公式即可，同时把`SIGNAL_LINE`

改成最后一条输出线的名字。注意三条红线：不用`:=`

；RSI 写`RSI(6)`

或`rsi.rsi1`

；分时均价用`SUM(AMOUNT,0)/SUM(VOL,0)/100`

，不要用`ZSTJJ`

。
