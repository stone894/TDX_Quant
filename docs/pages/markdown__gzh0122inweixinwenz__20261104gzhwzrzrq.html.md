---
title: "通达信TdxAiData：一次跑完沪深300 两融，四象限看清杠杆资金的真实动向 (opens new window)"
source_url: https://help.tdx.com.cn/quant/docs/markdown/gzh0122inweixinwenz/20261104gzhwzrzrq.html
fetched_at: 2026-11-06T06:02:52Z
---

# # 通达信TdxAiData：一次跑完沪深300 两融，四象限看清杠杆资金的真实动向 (opens new window)

通达信TdxAiData：Python独立数据接口，无需启动客户端 (opens new window)

📄

margin_analysis.py

取数据和输出数据图形和数据结论

```
# -*- coding: utf-8 -*-
"""
融资融券四象限分析（tdxaidata / tqs 真实数据）
股票池：沪深300成分股
数据窗口：最近 20 个已披露两融余额的交易日
"""
from __future__ import annotations
import json
from pathlib import Path
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
import pandas as pd
from tdxaidata import tqs
plt.rcParams["font.sans-serif"] = ["Microsoft YaHei", "SimHei"]
plt.rcParams["axes.unicode_minus"] = False
BASE = Path(__file__).resolve().parent
IMG_DIR = BASE.parent / "1027两融四象限-配图"
OUT_DIR = BASE
START = "2026-08-20"
END = "2026-10-23"
N_DAYS = 20
ORANGE = "#e74c3c"
BLUE = "#1e80ff"
GRAY = "#7f8c8d"
YELLOW = "#f39c12"
QUAD_COLORS = {
"价涨 + 融资增": ORANGE,
"价涨 + 融资降": YELLOW,
"价跌 + 融资增": BLUE,
"价跌 + 融资降": GRAY,
}
def to_series(records: list[dict], pos: int = 0) -> pd.Series:
idx = pd.to_datetime([r["Date"] for r in records], format="%Y%m%d")
return pd.Series([float(r["Value"][pos]) for r in records], index=idx)
def main() -> None:
IMG_DIR.mkdir(parents=True, exist_ok=True)
OUT_DIR.mkdir(parents=True, exist_ok=True)
# ---------- 1. 市场两融：以 SC01（融资/融券余额）的披露日为准 ----------
sc = tqs.get_scjy_value(["SC01", "SC25"], START, END)
m = pd.DataFrame(
{
"融资余额_万": to_series(sc["SC01"], 0),
"融券余额_万": to_series(sc["SC01"], 1),
}
).sort_index()
buy = to_series(sc["SC25"], 0).reindex(m.index)
m["融资买入额_万"] = buy
m = m.tail(N_DAYS).copy()
m["融资余额_亿"] = m["融资余额_万"] / 10000
m["融券余额_亿"] = m["融券余额_万"] / 10000
m["融资买入额_亿"] = m["融资买入额_万"] / 10000
m["余额日变化_亿"] = m["融资余额_亿"].diff()
idx_raw = tqs.get_market_data(
field_list=["Close"], stock_list=["999999.SH"], period="1d",
start_time=START, end_time=END, count=1000,
)["Close"]["999999.SH"]
idx_raw.index = pd.to_datetime(idx_raw.index).normalize()
m = m.join(idx_raw.rename("上证指数"), how="left")
first, last = m.iloc[0], m.iloc[-1]
mkt_change = last["融资余额_亿"] - first["融资余额_亿"]
mkt_change_pct = mkt_change / first["融资余额_亿"] * 100
short_change_pct = (last["融券余额_亿"] / first["融券余额_亿"] - 1) * 100
idx_change_pct = (last["上证指数"] / first["上证指数"] - 1) * 100
buy_sum = m["融资买入额_亿"].sum()
diffs = m["余额日变化_亿"].dropna()
up_day, down_day = diffs.idxmax(), diffs.idxmin()
streak = max_streak = 0
for v in diffs:
if v < 0:
streak += 1
max_streak = max(max_streak, streak)
else:
streak = 0
market_summary = {
"窗口": f"{first.name:%Y-%m-%d} ~ {last.name:%Y-%m-%d}",
"期初融资余额_亿": round(first["融资余额_亿"], 2),
"期末融资余额_亿": round(last["融资余额_亿"], 2),
"区间变化_亿": round(mkt_change, 2),
"区间变化率_%": round(mkt_change_pct, 2),
"融券余额_期初_亿": round(first["融券余额_亿"], 2),
"融券余额_期末_亿": round(last["融券余额_亿"], 2),
"融券余额变化率_%": round(short_change_pct, 2),
"上证区间涨跌_%": round(idx_change_pct, 2),
"区间融资买入额_亿": round(buy_sum, 2),
"单日最大增加": f"{up_day:%m-%d} +{diffs.max():.2f}亿",
"单日最大减少": f"{down_day:%m-%d} {diffs.min():.2f}亿",
"最长连续下降天数": max_streak,
}
# ---------- 2. 沪深300成分股两融 + 股价 ----------
codes = tqs.get_zzgz_stocklist("000300.SH", 0)
gp = tqs.get_gpjy_value(codes, ["GP03", "GP11", "GP12", "GP13"], START, END)
close = tqs.get_market_data(
field_list=["Close"], stock_list=codes, period="1d",
start_time=START, end_time=END, count=1000, dividend_type="front",
)["Close"]
close.index = pd.to_datetime(close.index).normalize()
name_map = {x["Code"]: x["Name"] for x in tqs.get_stock_list("5", 1)}
dates = m.index
rows = []
for code in codes:
d = gp.get(code, {})
bal_rec = d.get("GP03", [])
if not bal_rec:
continue
bal = to_series(bal_rec, 0).reindex(dates).dropna()
net = to_series(d.get("GP13", []), 0).reindex(dates).dropna()
px = close[code].reindex(dates).dropna()
if len(bal) < 2 or len(px) < 2:
continue
common = bal.index.intersection(px.index)
margin_chg = (bal.loc[common[-1]] / bal.loc[common[0]] - 1) * 100
px_chg = (px.loc[common[-1]] / px.loc[common[0]] - 1) * 100
if px_chg >= 0 and margin_chg >= 0:
quad = "价涨 + 融资增"
elif px_chg >= 0:
quad = "价涨 + 融资降"
elif margin_chg >= 0:
quad = "价跌 + 融资增"
else:
quad = "价跌 + 融资降"
rows.append(
{
"代码": code,
"名称": name_map.get(code, code),
"股价涨跌_%": round(px_chg, 2),
"融资余额变化_%": round(margin_chg, 2),
"融资净买入合计_亿": round(net.sum() / 10000, 2),
"期末融资余额_亿": round(bal.loc[common[-1]] / 10000, 2),
"象限": quad,
}
)
stocks = pd.DataFrame(rows)
valid = len(stocks)
quad_summary = (
stocks.groupby("象限")
.agg(
数量=("代码", "count"),
平均股价涨跌=("股价涨跌_%", "mean"),
平均融资变化=("融资余额变化_%", "mean"),
融资净买入合计_亿=("融资净买入合计_亿", "sum"),
)
.reindex(list(QUAD_COLORS))
.reset_index()
)
for col in ["平均股价涨跌", "平均融资变化", "融资净买入合计_亿"]:
quad_summary[col] = quad_summary[col].round(2)
top_increase = stocks.sort_values("融资余额变化_%", ascending=False).head(10)
top_decrease = stocks.sort_values("融资余额变化_%").head(10)
# ---------- 3. 输出 ----------
m.index.name = "日期"
m.to_csv(OUT_DIR / "市场两融_20日.csv", encoding="utf-8-sig")
stocks.sort_values("融资余额变化_%", ascending=False).to_csv(
OUT_DIR / "沪深300两融四象限_20日.csv", index=False, encoding="utf-8-sig"
)
quad_summary.to_csv(OUT_DIR / "四象限汇总.csv", index=False, encoding="utf-8-sig")
(OUT_DIR / "summary.json").write_text(
json.dumps(
{"market": market_summary, "valid": valid,
"quad": quad_summary.to_dict(orient="records")},
ensure_ascii=False, indent=2,
),
encoding="utf-8",
)
print(json.dumps(market_summary, ensure_ascii=False, indent=2))
print("\n有效样本：", valid)
print(quad_summary.to_string(index=False))
print("\n融资余额增幅前10：")
print(top_increase.to_string(index=False))
print("\n融资余额降幅前10：")
print(top_decrease.to_string(index=False))
# ---------- 4. 图 1：市场融资余额 vs 上证 ----------
fig, ax1 = plt.subplots(figsize=(11, 5.2), dpi=160)
ax1.plot(m.index, m["融资余额_亿"], color=ORANGE, lw=2.4, marker="o", ms=3, label="融资余额（亿元）")
ax1.set_ylabel("融资余额（亿元）", color=ORANGE)
ax1.tick_params(axis="y", labelcolor=ORANGE)
ax1.grid(alpha=0.2, ls="--")
ax2 = ax1.twinx()
ax2.plot(m.index, m["上证指数"], color=BLUE, lw=1.8, marker="s", ms=3, label="上证指数")
ax2.set_ylabel("上证指数", color=BLUE)
ax2.tick_params(axis="y", labelcolor=BLUE)
ax1.set_title(f"市场融资余额与上证指数（{first.name:%m-%d} ~ {last.name:%m-%d}）", fontsize=13)
fig.autofmt_xdate()
fig.tight_layout()
fig.savefig(IMG_DIR / "01市场融资余额与上证指数.png", bbox_inches="tight")
plt.close(fig)
# ---------- 5. 图 2：四象限散点 ----------
fig, ax = plt.subplots(figsize=(9.8, 6.8), dpi=160)
for quad, g in stocks.groupby("象限"):
ax.scatter(g["股价涨跌_%"], g["融资余额变化_%"], s=42, color=QUAD_COLORS[quad],
label=f"{quad}（{len(g)}只）", alpha=0.85, edgecolor="white", linewidth=0.5, zorder=3)
ax.axhline(0, color="#999", lw=1)
ax.axvline(0, color="#999", lw=1)
ax.set_xlabel("20 日股价涨跌幅（%）")
ax.set_ylabel("20 日融资余额变化率（%）")
ax.set_title(f"沪深300两融四象限（{first.name:%m-%d} ~ {last.name:%m-%d}，{valid}只）", fontsize=13)
ax.grid(alpha=0.2, ls="--")
ax.legend(fontsize=9, loc="upper left")
fig.tight_layout()
fig.savefig(IMG_DIR / "02沪深300四象限.png", bbox_inches="tight")
plt.close(fig)
# ---------- 6. 图 3：四象限数量 ----------
fig, ax = plt.subplots(figsize=(8.6, 4.8), dpi=160)
bars = ax.bar(quad_summary["象限"], quad_summary["数量"],
color=[QUAD_COLORS[q] for q in quad_summary["象限"]], alpha=0.9)
for b, v in zip(bars, quad_summary["数量"]):
ax.text(b.get_x() + b.get_width() / 2, v + 2, str(int(v)), ha="center", fontsize=12, fontweight="bold")
ax.set_ylabel("股票数量")
ax.set_title("沪深300成分股：融资余额变化四象限分布", fontsize=13)
ax.grid(axis="y", alpha=0.2, ls="--")
ax.set_ylim(0, max(quad_summary["数量"]) * 1.15)
fig.tight_layout()
fig.savefig(IMG_DIR / "03四象限分布.png", bbox_inches="tight")
plt.close(fig)
print("\n图片已输出：", IMG_DIR)
if __name__ == "__main__":
main()
```


1

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

通达信公式:融资数据验证统计


```
{市场}
融资余额:SCJYVALUE(1,1,0);
融券余额:SCJYVALUE(1,2,0);
融资买入额:SCJYVALUE(25,1,0);
融资UE:GPJYVALUE(3,1,0);
融券UV:GPJYVALUE(3,2,0);
{期末余额 -期初余额 = 期间净买入合计 - 首日净买入}
{期初余额+区间净买入（不含期初就期末）=期末余额}
净买入:GPJYVALUE(13,1,0);
当天融资UE:REF(融资UE,1)+净买入;
资买入:GPJYVALUE(11,1,0);
资卖出:GPJYVALUE(11,2,0);
DRS净买入:资买入-资卖出;
KSWZ:=REF(融资UE,19);
融资增:融资UE-KSWZ>0;
价涨:(C-REF(C,19))/REF(C,19)*100>0;
融资增%:(融资UE-KSWZ)/KSWZ*100;
价涨%:(C-REF(C,19))/REF(C,19)*100;
四象限:IF(融资增 AND 价涨,1,
IF(融资增=0 AND 价涨,2,
IF(融资增 AND 价涨=0,3,
4)));
```


1

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

← 20261030公众号文章 常见问题 →
