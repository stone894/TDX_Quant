---
title: "通达信TdxAiData：手搓一套自用的股软，字段/指标/规则/预警全搞定"
source_url: https://help.tdx.com.cn/quant/docs/markdown/gzh0122inweixinwenz/20261015gzhwz.html
fetched_at: 2026-11-06T06:03:59Z
---

# # 通达信TdxAiData：手搓一套自用的股软，字段/指标/规则/预警全搞定

**此功能为付费功能，详细使用过程请查看今天的公众号文章**通达信TdxAiData：手搓一套自用的股软，字段/指标/规则/预警全搞定 (opens new window)

📄

README.md

项目说明与快速开始

```
> **创建日期**：2026-10-15
# gusoft —— 用 TdxAiData 搭一套本地股票软件
不是行情脚本，是一套能天天用的本地网页股软：行情、K 线、板块、条件选股、历史回溯、自选预警、
每日收盘落库，全部数据来自 `tdxaidata`（`tqs`），全部存在本地 SQLite。
## 快速开始
```bash
pip install -r requirements.txt
python serve_ctl.py start # 启动服务，默认 http://127.0.0.1:8765
python serve_ctl.py stop # 停止
python warmup.py 120 # 首次运行：把各股票池日线灌进本地库（约 66 万行，几分钟）
python smoke_test.py # 14 个接口真实冒烟测试
python demo_run.py --bt 250 # 一次收盘后完整巡检，出日报 + CSV
```
浏览器打开 <http://127.0.0.1:8765>，八个页签：行情 / K线 / 分时 / 板块 / 强度榜 / 选股·回测 / 预警 / 自选管理。
## 目录结构
| 文件 | 作用 |
| --- | --- |
| `run_gusoft.py` | 一键启动入口 |
| `serve_ctl.py` | start/stop/restart/status（Windows 没有 taskkill，用 psutil 自管） |
| `warmup.py` | 预热本地日线库 |
| `demo_run.py` | 收盘巡检：行情 → 板块 → 预警 → 落库 → 扫描 → 回溯，出 `out/日报_*.md` |
| `smoke_test.py` | 全接口冒烟测试，打印真实耗时 |
| `bench_api.py` | 逐只 vs 批量 / 远程 vs 本地 压测，出 `out/性能对比.png` |
| `chart_backtest.py` | 六条规则回溯出图 |
| `gusoft/data.py` | `tqs` 唯一出口：日 K（本地优先）、批量快照、批量扩展、板块、交易日 |
| `gusoft/indicators.py` | MA/MACD/KDJ/RSI/BOLL/ATR/量比/20 日高低 |
| `gusoft/store.py` | SQLite：自选、收盘快照、预警、扫描历史、本地日线库 |
| `gusoft/scanner.py` | 六条选股规则 + 历史回溯 + 自选预警 |
| `gusoft/server.py` | 标准库 HTTP 服务 + JSON API（无 Flask、无框架） |
| `gusoft/web/index.html` | 前端单文件，Canvas 手绘 K 线 |
| `data/gusoft.db` | 本地数据库（唯一需要备份的文件） |
## API
| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/meta` | 版本、股票池、规则、本地库规模、最近交易日 |
| GET | `/api/quotes?pool=&top=` | 行情表（批量快照排序 + 只给显示行补指标） |
| GET | `/api/kline?code=&count=` | 日 K + 指标（多取 70 根预热） |
| GET | `/api/sectors?top=` | 560 板块涨跌榜 |
| GET | `/api/scan?rule=&pool=` | 条件选股 |
| GET | `/api/backtest?rule=&pool=&count=&hold=` | 规则历史回溯（含同期基准与超额） |
| GET | `/api/alerts?limit=` | 预警记录 |
| GET | `/api/watch` | 自选列表 |
| GET | `/api/export?type=snapshot` | 导出 CSV |
| POST | `/api/watch/add` / `/api/watch/del` | 加/删自选（`{"code":"600519","grp":"默认"}`） |
| POST | `/api/refresh` | 收盘巡检：自选预警 + 快照落库 |
## 六条选股规则
`ma_bull` 多头排列、`breakout` 放量突破 20 日高、`macd_gold` MACD 金叉、`oversold` 超跌反弹、
`limit_up` 涨停、`main_inflow` 主力净流入且站上 MA20。
回溯口径：T 日收盘出信号，T+hold 日收盘计收益，毛收益不含手续费与滑点；
同时算同期全池无条件平均作为基准，`超额 = 规则平均 - 基准`。
## 依赖说明
股软本体只需要 `tdxaidata` + `pandas` + `numpy`，HTTP 服务用标准库 `http.server`，
不装 Flask。`matplotlib`/`psutil`/`playwright` 只服务于压测出图、进程管理和截图。
## 数据与风险
- 行情为 tqs 前复权日 K + 实时快照，仅用于研究与盯盘。
- 回溯是事件研究，不是组合级回测：没有仓位管理、没有并发持仓限制、不含交易成本。
- 任何指标和信号都不构成投资建议。
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

📄

requirements.txt

依赖（股软本体只需 tdxaidata + pandas + numpy）

```
tdxaidata>=1.2.2
pandas>=2.0
numpy>=1.24
# 以下只为压测出图和截图服务，股软本体用不到
matplotlib>=3.7
psutil>=5.9
playwright>=1.40
```


1

2

3

4

5

6

7


2

3

4

5

6

7

📄

run_gusoft.py

一键启动入口

```
# -*- coding: utf-8 -*-
"""run_gusoft.py —— 一键启动股软。
用法：
python run_gusoft.py # 启动服务，默认 http://127.0.0.1:8765
python run_gusoft.py --port 9000 --open
"""
import argparse
import sys
import webbrowser
from gusoft.server import serve
def main() -> int:
ap = argparse.ArgumentParser(description="gusoft 本地股软（数据源：TdxAiData / tqs）")
ap.add_argument("--host", default="127.0.0.1")
ap.add_argument("--port", type=int, default=8765)
ap.add_argument("--open", action="store_true", help="启动后自动打开浏览器")
args = ap.parse_args()
url = f"http://{args.host}:{args.port}"
if args.open:
webbrowser.open(url)
serve(args.host, args.port)
return 0
if __name__ == "__main__":
sys.exit(main())
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

📄

serve_ctl.py

服务 start / stop / restart / status

```
# -*- coding: utf-8 -*-
"""serve_ctl.py —— 股软服务端进程管理（Windows 没有 taskkill，用 psutil 自己管）。
python serve_ctl.py start [--port 8765] 启动（后台）
python serve_ctl.py stop
python serve_ctl.py restart
python serve_ctl.py status
"""
import argparse
import os
import subprocess
import sys
import time
import psutil
HERE = os.path.dirname(os.path.abspath(__file__))
PIDFILE = os.path.join(HERE, "data", "gusoft.pid")
LOGFILE = os.path.join(HERE, "out_server.log")
def _running(pid: int) -> bool:
try:
p = psutil.Process(pid)
return "run_gusoft" in " ".join(p.cmdline())
except Exception:
return False
def start(port: int) -> int:
if os.path.exists(PIDFILE):
try:
pid = int(open(PIDFILE).read().strip())
if _running(pid):
print(f"[serve_ctl] 已在运行 pid={pid}，先 stop 再 start")
return 0
except Exception:
pass
os.makedirs(os.path.dirname(PIDFILE), exist_ok=True)
log = open(LOGFILE, "a", encoding="utf-8")
env = dict(os.environ, PYTHONIOENCODING="utf-8")
proc = subprocess.Popen([sys.executable, os.path.join(HERE, "run_gusoft.py"), "--port", str(port)],
cwd=HERE, stdout=log, stderr=subprocess.STDOUT, env=env)
open(PIDFILE, "w").write(str(proc.pid))
time.sleep(2)
print(f"[serve_ctl] 已启动 pid={proc.pid} → http://127.0.0.1:{port} 日志 {LOGFILE}")
return 0
def stop() -> int:
n = 0
for p in psutil.process_iter(["pid", "cmdline"]):
cl = " ".join(p.info["cmdline"] or [])
if "run_gusoft" in cl:
try:
p.terminate()
n += 1
except Exception:
pass
if os.path.exists(PIDFILE):
os.remove(PIDFILE)
print(f"[serve_ctl] 已停止 {n} 个进程")
return 0
def status() -> int:
found = [(p.info["pid"], " ".join(p.info["cmdline"] or [])[:70])
for p in psutil.process_iter(["pid", "cmdline"]) if "run_gusoft" in " ".join(p.info["cmdline"] or [])]
print("[serve_ctl] 运行中:" if found else "[serve_ctl] 未运行")
for pid, cl in found:
print(f" pid={pid} {cl}")
return 0
def main() -> int:
ap = argparse.ArgumentParser()
ap.add_argument("action", choices=["start", "stop", "restart", "status"])
ap.add_argument("--port", type=int, default=8765)
a = ap.parse_args()
return {"start": lambda: start(a.port), "stop": stop, "status": status,
"restart": lambda: (stop(), time.sleep(1), start(a.port))}[a.action]()
if __name__ == "__main__":
sys.exit(main())
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

📄

warmup.py

首次预热本地日线库

```
# -*- coding: utf-8 -*-
"""warmup.py —— 首次运行把整池日线灌进本地 SQLite 库，之后页面与扫描都走本地。
用法：python warmup.py [条数]
实测：沪深300 × 120 根日线，取数约 13 秒，之后同类查询毫秒级返回。
"""
import sys
import time
from gusoft import data as dm
from gusoft import indicators as ind
from gusoft import scanner as sc
from gusoft import store
def main() -> int:
count = int(sys.argv[1]) if len(sys.argv) > 1 else 120
store.init_db()
for pool, (name, _spec) in sc.POOLS.items():
codes = sc.pool_codes(pool)
t0 = time.time()
k = dm.data.kline(codes, count=count)
n = len(k)
print(f"[warmup] {name:<6} {len(codes):>4} 只 × {count} 根 → 本地 {n:>6} 行，用时 {time.time() - t0:5.1f}s")
print(f"[warmup] 本地日线库合计 {store.kline_count()} 行 / {len(store.kline_days())} 个交易日，"
f"最近交易日 {dm.data.last_trading_day()}")
return 0
if __name__ == "__main__":
sys.exit(main())
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

📄

demo_run.py

收盘巡检：行情→板块→预警→落库→扫描→回溯，出 out/日报

```
# -*- coding: utf-8 -*-
"""demo_run.py —— 一次「收盘后完整巡检」，把股软的四件事真跑一遍并出报告。
python demo_run.py # 默认沪深300，自选 = 脚本内置几只票
python demo_run.py --pool zz1000 # 换股票池
python demo_run.py --hold 10 # 回溯持有期
输出：
终端日报 + out/日报_YYYY-MM-DD.md + out/扫描_*.csv + out/快照_YYYY-MM-DD.csv
"""
from __future__ import annotations
import argparse
import os
import sys
import time
from datetime import date
from gusoft import data as dm
from gusoft import indicators as ind
from gusoft import scanner as sc
from gusoft import store
OUT = os.path.join(os.path.dirname(os.path.abspath(__file__)), "out")
DEFAULT_WATCH = ["600519.SH", "300750.SZ", "002493.SZ", "601318.SH", "688981.SH", "000858.SZ"]
def line(t: str = "-", n: int = 78):
print(t * n)
def main() -> int:
ap = argparse.ArgumentParser()
ap.add_argument("--pool", default="hs300")
ap.add_argument("--count", type=int, default=120, help="扫描用K线根数")
ap.add_argument("--bt", type=int, default=250, help="回溯窗口交易日数")
ap.add_argument("--hold", type=int, default=5)
ap.add_argument("--top", type=int, default=10)
args = ap.parse_args()
os.makedirs(OUT, exist_ok=True)
store.init_db()
today = date.today().strftime("%Y-%m-%d")
pool_name = sc.POOLS[args.pool][0]
# 0. 自选股：库里没有就写入内置几只（真实写入 SQLite）
have = {w["code"] for w in store.list_watch()}
names = dm.data.names(DEFAULT_WATCH)
for code in DEFAULT_WATCH:
if code not in have:
store.add_watch(code, names.get(code, code), grp="演示")
watch = [w["code"] for w in store.list_watch()]
print(f"\n[gusoft demo] {today} {pool_name} · 数据源 tdxaidata {__import__('tdxaidata').__version__}")
line("=")
print(f"自选股 {len(watch)} 只: {'、'.join(names.get(c, c) for c in watch)}")
print(f"本地日线库: {store.kline_count()} 行 / {len(store.kline_days())} 个交易日")
# 1. 全池行情
t0 = time.time()
codes = sc.pool_codes(args.pool)
snaps = dm.data.snapshots(codes)
mores = dm.data.more_batch(codes)
zaf = [q["zaf"] for q in snaps.values()]
up, down = sum(1 for v in zaf if v > 0), sum(1 for v in zaf if v < 0)
net = sum(m.get("zjl_main") or 0 for m in mores.values())
net_buy = sum(m.get("zjl_buy") or 0 for m in mores.values())
print(f"\n【1】{pool_name} 行情：{len(snaps)} 只，上涨 {up} / 下跌 {down}，"
f"主力净流入合计 {net / 10000:.1f} 亿元 / 主买净额合计 {net_buy / 10000:.1f} 亿元，"
f"耗时 {time.time() - t0:.2f}s")
top = sorted(snaps.items(), key=lambda kv: kv[1]["zaf"], reverse=True)[:args.top]
nm = dm.data.names([c for c, _ in top])
print(f" 涨幅前 {args.top}：" + "、".join(f"{nm.get(c, c)} {q['zaf']:+.2f}%" for c, q in top))
# 2. 板块
t0 = time.time()
sup, sdn = dm.data.sector_board(6)
print(f"\n【2】板块（{len(dm.data.sectors())} 个指数，批量快照）耗时 {time.time() - t0:.2f}s")
print(" 领涨：" + "、".join(f"{r['name']} {r['zaf']:+.2f}%" for r in sup))
print(" 领跌：" + "、".join(f"{r['name']} {r['zaf']:+.2f}%" for r in sdn))
# 3. 自选股预警
t0 = time.time()
wk = dm.data.kline(watch, count=25)
frames = sc.enrich_pool(wk)
alerts = sc.check_alerts(frames, names, watch)
for a in alerts:
store.add_alert(a["code"], a["name"], a["rule"], a["level"], a["price"], a["msg"])
print(f"\n【3】自选股预警：命中 {len(alerts)} 条，耗时 {time.time() - t0:.2f}s")
for a in alerts[:10]:
print(f" [{a['level']:<4}] {a['name']}({a['code']}) {a['rule']:<8} {a['price']} {a['msg']}")
for code in watch:
s = ind.last_snapshot(frames[code])
if s:
print(f" · {names.get(code, code)} 收 {s['close']}（{s['pct_chg']:+.2f}%）"
f" MA20={s['ma20']} RSI14={s['rsi14']} 量比={s['vol_ratio']}")
# 4. 收盘快照落库
day = store.kline_days()[-1] if store.kline_days() else today
rows = []
for code, df in frames.items():
s = ind.last_snapshot(df)
m = mores.get(code, {})
rows.append({"code": code, "name": names.get(code, code), "close": s["close"],
"pct_chg": s["pct_chg"], "volume": float(df["volume"].iloc[-1]),
"hsl": m.get("hsl"), "zjl": m.get("zjl_buy"), "zjl_main": m.get("zjl_main"),
"ltsz": m.get("ltsz"), "pe": m.get("pe"), "vol_ratio": s["vol_ratio"]})
saved = store.save_snapshot(day, rows)
print(f"\n【4】收盘快照落库：{saved} 行 → {store.DB_PATH}（day={day}）")
# 5. 六条规则扫描 + 回溯（扫描用 count 根，回溯另取 bt 根，两者口径不同要分开取）
line()
k = dm.data.kline(codes, count=args.count)
k_bt = k if args.bt <= args.count else dm.data.kline(codes, count=args.bt)
bt_days = max(0, len(k_bt["date"].unique()) - 30 - args.hold)
print(f"【5】条件扫描与历史回溯（{pool_name} {len(codes)} 只，扫描用 {args.count} 根日K，"
f"回溯 {len(k_bt['date'].unique())} 根日K / {bt_days} 个信号日 / 持有 {args.hold} 日）")
print(f" {'规则':<12}{'今日命中':>7}{'样本':>8}{'平均':>8}{'超额':>8}{'胜率':>7}{'基准':>7}{'基准胜率':>9}{'最好':>8}{'最差':>8}")
report = [f"【5】条件扫描与历史回溯（{pool_name} {len(codes)} 只，扫描用 {args.count} 根日K，"
f"回溯 {len(k_bt['date'].unique())} 根日K / {bt_days} 个信号日 / 持有 {args.hold} 日，"
f"基准=同期全池无条件平均）",
"| 规则 | 今日命中 | 样本 | 平均收益% | 超额% | 胜率% | 基准平均% | 基准胜率% | 最好% | 最差% |",
"|---|---|---|---|---|---|---|---|---|---|"]
for rule in sc.RULES:
t0 = time.time()
more_map = mores if rule == "main_inflow" else None
hits = sc.screen(codes, rule, k, more_map)
bt = sc.backtest(codes, rule, k_bt, hold=args.hold, min_period=30, more_map=more_map)
if bt.get("samples"):
report.append(f"| {rule} | {len(hits)} | {bt['samples']} | {bt['avg']} | {bt['excess']} | "
f"{bt['win_rate']} | {bt['bench_avg']} | {bt['bench_win_rate']} | {bt['best']} | {bt['worst']} |")
print(f" {rule:<12}{len(hits):>7}{bt['samples']:>8}{bt['avg']:>8}{bt['excess']:>8}"
f"{bt['win_rate']:>7}{bt['bench_avg']:>7}{bt['bench_win_rate']:>9}{bt['best']:>8}{bt['worst']:>8}"
f" ({time.time() - t0:.1f}s)")
top_hits = hits[:args.top]
store.add_scan(args.pool, rule, len(hits), top_hits)
if top_hits:
hit_names = dm.data.names([h["code"] for h in top_hits])
path = os.path.join(OUT, f"扫描_{rule}_{today}.csv")
with open(path, "w", encoding="utf-8-sig", newline="") as f:
f.write("code,name,close,pct_chg,vol_ratio,ma5,ma20,rsi14,dif,dea\n")
for h in top_hits:
f.write(f"{h['code']},{hit_names.get(h['code'], h['code'])},{h.get('close')},"
f"{h.get('pct_chg')},{h.get('vol_ratio')},{h.get('ma5')},{h.get('ma20')},"
f"{h.get('rsi14')},{h.get('dif')},{h.get('dea')}\n")
else:
print(f" {rule:<10}{len(hits):>8}{'样本不足':>10}")
line("=")
# 6. 导出 CSV
snap_csv = os.path.join(OUT, f"快照_{today}.csv")
from gusoft.server import api_export
with open(snap_csv, "w", encoding="utf-8") as f:
f.write(api_export("snapshot"))
md = os.path.join(OUT, f"日报_{today}.md")
with open(md, "w", encoding="utf-8") as f:
f.write(f"# gusoft 收盘巡检 {today}\n\n")
f.write(f"- 数据源：tdxaidata / tqs；股票池：{pool_name}（{len(codes)} 只）\n")
f.write(f"- 上涨 {up} / 下跌 {down}；主力净额合计 {net / 10000:.2f} 亿元\n")
f.write(f"- 自选 {len(watch)} 只，本日预警 {len(alerts)} 条\n")
f.write(f"- tqs 累计请求 {dm.CALL_STATS['requests']} 次，累计耗时 {dm.CALL_STATS['seconds']:.1f}s\n\n")
f.write("\n".join(report) + "\n")
print(f"[gusoft demo] 报告 {md}")
print(f"[gusoft demo] 快照 {snap_csv}")
return 0
if __name__ == "__main__":
sys.exit(main())
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

📄

smoke_test.py

全接口真实冒烟测试

```
# -*- coding: utf-8 -*-
"""smoke_test.py —— 股软全接口冒烟测试：每个接口真实调一次，打印真实耗时与关键结果。
python smoke_test.py # 用本地库
python smoke_test.py --cold # 先清服务内存缓存再测（观察真实计算耗时）
输出即文章里引用的性能数字来源，不接受手写。
"""
from __future__ import annotations
import argparse
import json
import time
import urllib.error
import urllib.request
BASE = "http://127.0.0.1:8765"
def get(path: str):
t0 = time.time()
with urllib.request.urlopen(BASE + path, timeout=900) as r:
data = json.loads(r.read().decode("utf-8"))
return round(time.time() - t0, 2), data
def post(path: str, body: dict):
t0 = time.time()
rq = urllib.request.Request(BASE + path, data=json.dumps(body).encode("utf-8"),
headers={"Content-Type": "application/json"})
with urllib.request.urlopen(rq, timeout=900) as r:
data = json.loads(r.read().decode("utf-8"))
return round(time.time() - t0, 2), data
def main() -> int:
ap = argparse.ArgumentParser()
ap.add_argument("--cold", action="store_true", help="忽略服务侧 10 秒内存缓存")
a = ap.parse_args()
print(f"=== gusoft 冒烟测试 {time.strftime('%Y-%m-%d %H:%M:%S')} → {BASE} ===")
s, d = get("/api/meta")
print(f"[meta] {s}s tdxaidata={d['data_api']} 本地日线 {d['local_bars']} 行 / {d['local_days']} 个交易日"
f" 最近交易日={d['last_trading_day']} 自选={d['watch']} 股票池={list(d['pools'])}")
p = "/api/quotes?pool=hs300&top=30" + ("&nocache=1" if a.cold else "")
s, d = get(p)
print(f"[quotes 沪深300] {s}s n={d['n']} 显示={d['shown']} 涨{d['up']}/跌{d['down']}/平{d['flat']} "
f"主力净额={d['net_zjl_main']:.0f}万 主买净额={d['net_zjl']:.0f}万")
print(f" 首行 {d['rows'][0]['name']}({d['rows'][0]['code']}) {d['rows'][0]['zaf']:+.2f}% "
f"MA20={d['rows'][0]['ma20']} 量比={d['rows'][0]['vol_ratio']}")
s, d = get("/api/kline?code=002493.SZ&count=120")
b = d["bars"]
print(f"[kline 002493.SZ] {s}s {d['name']} {len(b)} 根 首根MA60={b[0]['ma60']} "
f"(预热后有值) 末根量比={b[-1]['vol_ratio']}")
s, d = get("/api/minute?code=002493.SZ")
print(f"[minute 002493.SZ] {s}s {d['name']} {len(d['price'])} 根分钟线 "
f"现价={d.get('last')} ({d.get('pct')}%) 最高={d.get('high')} 最低={d.get('low')}")
s, d = get("/api/sectors?top=6")
print(f"[sectors 560个指数] {s}s 涨: {'、'.join(r['name'] for r in d['up'])}")
print(f" 跌: {'、'.join(r['name'] for r in d['down'])}")
for rule in ("ma_bull", "macd_gold", "oversold", "main_inflow"):
s, d = get(f"/api/scan?rule={rule}&pool=hs300")
print(f"[scan {rule}] {s}s 命中 {len(d['hits'])} 只 例: "
+ "、".join(h["name"] for h in d["hits"][:3]))
for rule in ("ma_bull", "limit_up", "main_inflow"):
s, d = get(f"/api/backtest?rule={rule}&pool=hs300&count=250&hold=5")
print(f"[backtest {rule}] {s}s 样本={d['samples']} 平均={d['avg']}% 超额={d['excess']}% "
f"胜率={d['win_rate']}% 基准={d['bench_avg']}% 最好={d['best']}% 最差={d['worst']}%")
s, d = post("/api/watch/add", {"code": "601899", "grp": "冒烟"})
print(f"[watch/add] {s}s {d}")
s, d = get("/api/watch")
print(f"[watch 自选列表] {s}s 共 {len(d['rows'])} 只 例: "
+ "、".join(f"{r['name']}({r['grp']})" for r in d["rows"][:3]))
s, d = post("/api/watch", {"code": "601899", "grp": "冒烟"}) # 旧接口兼容
print(f"[watch 旧接口add] {s}s {d}")
s, d = post("/api/watch_del", {"code": "601899"}) # 旧接口兼容
print(f"[watch_del 旧接口] {s}s {d}")
s, d = post("/api/watch/del", {"code": "601899.SH"})
print(f"[watch/del] {s}s {d}")
s, d = post("/api/refresh", {})
print(f"[refresh 收盘巡检] {s}s day={d['day']} 预警={d['alerts_saved']} 快照={d['snapshot_saved']} 行")
s, d = get("/api/alerts?limit=5")
print(f"[alerts] {s}s 共 {len(d['rows'])} 条 例: "
+ "；".join(f"{r['name']} {r['rule']}" for r in d["rows"][:3]))
t0 = time.time()
with urllib.request.urlopen(BASE + "/api/export?type=snapshot", timeout=300) as r:
csv = r.read().decode("utf-8")
print(f"[export CSV] {round(time.time() - t0, 2)}s {len(csv.splitlines())} 行")
print("=== 全部通过：15 个接口 ===")
print(" GET /api/meta /api/quotes /api/kline /api/minute /api/sectors /api/scan "
"/api/backtest /api/alerts /api/watch /api/export")
print(" POST /api/watch/add /api/watch/del /api/watch（旧） /api/watch_del（旧） /api/refresh")
return 0
if __name__ == "__main__":
try:
raise SystemExit(main())
except urllib.error.URLError as e:
print(f"[smoke] 连不上 {BASE}：{e} 先跑 python serve_ctl.py start")
raise SystemExit(1)
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

📄

bench_api.py

逐只 vs 批量 / 远程 vs 本地 压测

```
# -*- coding: utf-8 -*-
"""bench_api.py —— 真实压测：把「逐只循环」和「1.1.1 批量接口」放在一起跑，输出对比表和柱状图。
python bench_api.py # 快照/扩展信息/板块三组对比 + 出图
python bench_api.py --codes 100 # 股票池取前 100 只（逐太慢时用）
为什么必须自己测：文档只写「支持批量」，没人告诉你差多少。股软页面要 15 秒自动刷新，
差 70 倍就是能上与不能上的区别。
"""
from __future__ import annotations
import argparse
import os
import time
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
from tdxaidata import tqs
import gusoft.data as dm
import gusoft.store as store
OUT = os.path.join(os.path.dirname(os.path.abspath(__file__)), "out")
plt.rcParams["font.sans-serif"] = ["Microsoft YaHei", "SimHei"]
plt.rcParams["axes.unicode_minus"] = False
def timed(fn, *a, **kw):
t0 = time.time()
res = fn(*a, **kw)
return round(time.time() - t0, 2), res
def bench_snapshot(codes: list[str]) -> dict:
fields = ["Now", "LastClose", "Open", "High", "Low", "Volume", "Amount", "Average"]
t_loop, n1 = timed(lambda: [tqs.get_market_snapshot(c, fields) for c in codes])
t_batch, n2 = timed(lambda: tqs.get_market_snapshot_batch(codes, fields))
return {"项目": f"行情快照 {len(codes)} 只", "逐只": t_loop, "批量": t_batch,
"倍数": round(t_loop / t_batch, 1) if t_batch else 0, "条数": len(n2 or {})}
def bench_more(codes: list[str]) -> dict:
fields = ["ZJL", "HSL", "PE", "LTGZ", "ZDF5", "ZDF20", "ZTPRICE", "ZTCOUNT", "RFA"]
t_loop, _ = timed(lambda: [tqs.get_more_info(c, fields) for c in codes])
t_batch, n2 = timed(lambda: tqs.get_more_info_batch(codes, fields))
return {"项目": f"扩展信息 {len(codes)} 只", "逐只": t_loop, "批量": t_batch,
"倍数": round(t_loop / t_batch, 1) if t_batch else 0, "条数": len(n2 or {})}
def bench_sector() -> dict:
secs = dm.data.sectors() # 560 个板块指数
codes = [s["code"] for s in secs]
t_loop, _ = timed(lambda: [tqs.get_market_snapshot(c, ["Now", "LastClose", "Close"])
for c in codes])
t_batch, n2 = timed(lambda: tqs.get_market_snapshot_batch(codes, ["Now", "LastClose", "Close"]))
return {"项目": f"板块快照 {len(codes)} 个", "逐只": t_loop, "批量": t_batch,
"倍数": round(t_loop / t_batch, 1) if t_batch else 0, "条数": len(n2 or {})}
def bench_kline(codes: list[str], count: int = 120) -> dict:
t_remote, _ = timed(lambda: dm.data.kline(codes, count=count, use_local=False))
t_local, _ = timed(lambda: dm.data.kline(codes, count=count))
return {"项目": f"日K {len(codes)} 只×{count}根", "逐只": t_remote, "批量": t_local,
"倍数": round(t_remote / t_local, 1) if t_local else 0, "条数": count * len(codes)}
def draw(rows: list[dict], path: str) -> None:
labels = [r["项目"] for r in rows]
a = [max(r["逐只"], 0.001) for r in rows]
b = [max(r["批量"], 0.001) for r in rows]
import numpy as np
x = np.arange(len(rows))
fig, ax = plt.subplots(figsize=(9, 4.2), dpi=160)
ax.bar(x - 0.19, a, 0.38, label="旧写法：逐只循环", color="#c0392b")
ax.bar(x + 0.19, b, 0.38, label="gusoft：批量接口 / 本地库", color="#ff6b3d")
ax.set_yscale("log")
ax.set_ylim(top=max(a) * 2.3) # 预留头顶空间，避免「倍数」标注顶到标题
ax.set_xticks(x)
ax.set_xticklabels(labels, fontsize=9, linespacing=1.5)
ax.set_ylabel("耗时（秒，对数轴）")
ax.set_title("同一份数据，两种写法：gusoft 实测对比", pad=14)
for i, r in enumerate(rows):
ax.text(i - 0.19, a[i] * 1.10, f"{a[i]}s", ha="center", fontsize=8, color="#c0392b")
ax.text(i + 0.19, b[i] * 1.10, f"{b[i]}s", ha="center", fontsize=8, color="#ff6b3d")
ax.text(i, a[i] * 1.5, f"{r['倍数']}倍", ha="center", fontsize=9, color="#333")
ax.legend(fontsize=9, loc="upper right")
ax.grid(axis="y", alpha=.25)
fig.tight_layout()
# 自检：x 轴标签互相压住的话，图底部会看着像两行字摞在一起
fig.canvas.draw()
r = fig.canvas.get_renderer()
boxes = [(t.get_text(), t.get_window_extent(r)) for t in ax.get_xticklabels() if t.get_text().strip()]
bad = sum(1 for i in range(len(boxes)) for j in range(i + 1, len(boxes))
if min(boxes[i][1].x1, boxes[j][1].x1) - max(boxes[i][1].x0, boxes[j][1].x0) > 1
and min(boxes[i][1].y1, boxes[j][1].y1) - max(boxes[i][1].y0, boxes[j][1].y0) > 1)
# 自检 2：「倍数」标注是否冲出坐标区顶部或压住标题
from matplotlib.transforms import Bbox
ax_bb = ax.get_window_extent(r)
title_bb = ax.title.get_window_extent(r)
over = sum(1 for t in ax.texts
if t.get_window_extent(r).y1 > ax_bb.y1 + 1
or Bbox.intersection(t.get_window_extent(r), title_bb) is not None)
print(f"[bench] 底部文字重叠 {bad} 处，头顶标注越界/压标题 {over} 处"
+ ("（正常）" if (bad == 0 and over == 0) else "（需修）"))
fig.savefig(path)
print(f"[bench] 图表 {path}")
def main() -> int:
ap = argparse.ArgumentParser()
ap.add_argument("--codes", type=int, default=300)
a = ap.parse_args()
os.makedirs(OUT, exist_ok=True)
store.init_db()
codes = dm.data.stock_pool("23")
codes = [r["code"] for r in codes][: a.codes]
rows = [bench_snapshot(codes), bench_more(codes), bench_sector(),
bench_kline(codes[:100], 120)]
print(f"\n=== gusoft 接口压测（tds {__import__('tdxaidata').__version__}，"
f"{time.strftime('%Y-%m-%d %H:%M:%S')}）===")
print(f"{'项目':<22}{'逐只/远程(s)':>14}{'批量/本地(s)':>14}{'加速':>9}{'数据量':>10}")
for r in rows:
print(f"{r['项目']:<22}{r['逐只']:>14}{r['批量']:>14}{str(r['倍数']) + '倍':>9}{r['条数']:>10}")
draw(rows, os.path.join(OUT, "性能对比.png"))
with open(os.path.join(OUT, "性能对比.txt"), "w", encoding="utf-8") as f:
for r in rows:
f.write(f"{r['项目']}\t{r['逐只']}\t{r['批量']}\t{r['倍数']}\n")
return 0
if __name__ == "__main__":
raise SystemExit(main())
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

📄

chart_backtest.py

六条规则回溯出图

```
# -*- coding: utf-8 -*-
"""chart_backtest.py —— 把六条规则的真实回溯结果画成图（数据来自 gusoft 自己的回溯函数，不是编的）。
python chart_backtest.py --pool hs300 --count 250 --hold 5
"""
from __future__ import annotations
import argparse
import os
import re
import time
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
import gusoft.data as dm
import gusoft.scanner as sc
import gusoft.store as store
OUT = os.path.join(os.path.dirname(os.path.abspath(__file__)), "out")
plt.rcParams["font.sans-serif"] = ["Microsoft YaHei", "SimHei"]
plt.rcParams["axes.unicode_minus"] = False
def _wrap_cjk(s: str, width: int = 4) -> list[str]:
"""按固定字数折行；最后一行只剩 1 个字就并回上一行，避免出现孤字。"""
rows = [s[i:i + width] for i in range(0, len(s), width)] or [s]
if len(rows) > 1 and len(rows[-1]) == 1:
rows[-2] += rows[-1]
rows.pop()
return rows
def short_label(rule: str, desc: str) -> str:
"""x 轴标签：规则代号一行 + 中文短名折行。
原来直接把 sc.RULES 里的整句条件（例：均线多头排列（MA5>MA10>MA20 且价在MA20上方））
塞进 x 轴，6 个标签横向互相压在一起，最宽处叠了将近 300 像素，底部看着就是两行字摞在一起。
条件全文留在正文表格里，图上只放代号和短名。
"""
desc = re.split(r"[（(]", desc)[0].strip()
return "\n".join([rule] + _wrap_cjk(desc))
def check_overlap(fig, tag: str = "") -> int:
"""渲染后量一遍文字包围盒，报告互相压住的行数（自检，防止以后又叠回去）。"""
fig.canvas.draw()
r = fig.canvas.get_renderer()
items = []
for ax in fig.axes:
for lbl in ax.get_xticklabels():
if lbl.get_text().strip():
items.append((lbl.get_text().replace("\n", "/")[:16], lbl.get_window_extent(r)))
bad = 0
for i in range(len(items)):
for j in range(i + 1, len(items)):
a, b = items[i][1], items[j][1]
ox = min(a.x1, b.x1) - max(a.x0, b.x0)
oy = min(a.y1, b.y1) - max(a.y0, b.y0)
if ox > 1 and oy > 1:
bad += 1
if bad <= 3:
print(f"[chart] {tag} 标签重叠: {items[i][0]!r} × {items[j][0]!r} "
f"{ox:.0f}×{oy:.0f}px")
print(f"[chart] {tag} 底部文字重叠 {bad} 处" + ("（正常）" if bad == 0 else "（需修）"))
return bad
def main() -> int:
ap = argparse.ArgumentParser()
ap.add_argument("--pool", default="hs300")
ap.add_argument("--count", type=int, default=250)
ap.add_argument("--hold", type=int, default=5)
a = ap.parse_args()
os.makedirs(OUT, exist_ok=True)
store.init_db()
codes = sc.pool_codes(a.pool)
k = dm.data.kline(codes, count=a.count)
more = dm.data.more_batch(codes)
stats = []
for rule in sc.RULES:
t0 = time.time()
s = sc.backtest(codes, rule, k, hold=a.hold, more_map=more if rule == "main_inflow" else None)
s["seconds"] = round(time.time() - t0, 1)
stats.append(s)
print(f"[bt] {rule:<12} 样本={s.get('samples')} 平均={s.get('avg')} 超额={s.get('excess')} "
f"胜率={s.get('win_rate')} ({s['seconds']}s)")
names = [short_label(s["rule"], sc.RULES.get(s["rule"], "")) for s in stats]
avg = [s.get("avg", 0) for s in stats]
exc = [s.get("excess", 0) for s in stats]
win = [s.get("win_rate", 0) for s in stats]
sam = [s.get("samples", 0) for s in stats]
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(13, 5.2), dpi=160, gridspec_kw={"width_ratios": [1.35, 1]})
x = range(len(stats))
ax1.bar([i - .2 for i in x], avg, .4, label="信号后5日平均收益", color="#ff6b3d")
ax1.bar([i + .2 for i in x], exc, .4, label="超额（减同期基准）", color="#2d7ff9")
ax1.axhline(0, color="#666", lw=.8)
ax1.set_xticks(list(x))
ax1.set_xticklabels(names, fontsize=9, linespacing=1.5)
ax1.set_ylabel("平均收益 %")
ax1.set_title(f"{sc.POOLS[a.pool][0]} 近{a.count}个交易日 · 持有{a.hold}日 · 事件研究")
ax1.legend(fontsize=8)
ax1.grid(axis="y", alpha=.25)
ax2.bar(x, win, .5, color="#21c17a", label="规则胜率")
ax2.axhline(stats[0].get("bench_win_rate", 45), color="#c0392b", ls="--", lw=1,
label=f"全池基准胜率 {stats[0].get('bench_win_rate')}%")
for i, s in enumerate(sam):
ax2.text(i, win[i] + .6, f"{s}", ha="center", fontsize=7, color="#666")
ax2.set_xticks(list(x))
ax2.set_xticklabels(names, fontsize=9, linespacing=1.5)
ax2.set_ylabel("胜率 %")
ax2.set_ylim(0, max(win) * 1.25)
ax2.set_title("胜率对比（柱顶数字=样本数）")
ax2.legend(fontsize=8)
ax2.grid(axis="y", alpha=.25)
fig.tight_layout()
check_overlap(fig, "回溯图")
path = os.path.join(OUT, f"回溯结果_{a.pool}_{a.count}_{a.hold}.png")
fig.savefig(path)
print(f"[chart] {path}")
return 0
if __name__ == "__main__":
raise SystemExit(main())
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

📄

gusoft/init.py

包入口

```
# -*- coding: utf-8 -*-
"""gusoft —— 用 tdxaidata(tqs) 数据接口搭的本地网页版股票软件。"""
__version__ = "1.0.0"
```


1

2

3


2

3

📄

gusoft/data.py

tqs 唯一出口：日K（本地优先）、批量快照、批量扩展、板块、交易日

```
# -*- coding: utf-8 -*-
"""data.py —— 股软数据层：把 tdxaidata(tqs) 的零散接口封装成「行情表 / K线 / 板块 / 资金」四类可复用数据服务。
设计要点
1. 只依赖 tdxaidata + pandas，不引入其它三方库，方便读者照抄。
2. 所有接口都做「空值兜底」：tqs 新版出错时打印 [错误] 并返回 None/{}，这里统一返回空 DataFrame，调用方不必到处判空。
3. 带 TTL 内存缓存 + 线程锁：网页 5 秒轮询不会把行情服务器打爆，扫描器重复取同一批 K 线也不重复请求。
"""
from __future__ import annotations
import threading
import time
from datetime import datetime
from typing import Any, Iterable
import pandas as pd
from tdxaidata import tqs
from . import store
FIELDS_DAILY = ["Open", "High", "Low", "Close", "Volume", "Amount"]
# --------------------------------------------------------------------------- #
# 工具
# --------------------------------------------------------------------------- #
def f2v(value: Any, default: float = 0.0) -> float:
"""DLL 返回的数值几乎都是字符串，这里统一转 float，脏值给默认值。"""
try:
if value is None or value == "":
return default
return float(value)
except (TypeError, ValueError):
return default
class TTLCache:
"""极简 TTL 缓存：一个键一个过期时间戳，够用且不引三方依赖。"""
def __init__(self) -> None:
self._d: dict[str, tuple[float, Any]] = {}
self._lock = threading.Lock()
def get(self, key: str):
with self._lock:
hit = self._d.get(key)
if hit and time.time() - hit[0] < hit[1][0]:
return hit[1][1]
return None
def set(self, key: str, value: Any, ttl: float) -> None:
with self._lock:
self._d[key] = (time.time(), (ttl, value))
def clear(self) -> None:
with self._lock:
self._d.clear()
_cache = TTLCache()
_snap_lock = threading.Lock() # tqs 内部连接不保证并发安全，取行情串行化
CALL_STATS = {"requests": 0, "seconds": 0.0}
def _t() -> float:
return time.perf_counter()
def _mark(start: float) -> None:
CALL_STATS["requests"] += 1
CALL_STATS["seconds"] += _t() - start
# --------------------------------------------------------------------------- #
# 数据服务
# --------------------------------------------------------------------------- #
class MarketData:
"""股软唯一的数据入口。"""
# ---------- 股票池 ---------- #
def stock_pool(self, market: str = "23") -> list[dict]:
"""股票池：market 见官方取值表（5=全部A股 23=沪深300 50=沪深A股 ...）。"""
key = f"pool:{market}"
cached = _cache.get(key)
if cached is not None:
return cached
start = _t()
rows = tqs.get_stock_list(market=market, list_type=1) or []
rows = [{"code": r.get("Code", ""), "name": r.get("Name", "")} for r in rows if r.get("Code")]
_mark(start)
_cache.set(key, rows, ttl=3600)
return rows
def names(self, codes: Iterable[str]) -> dict[str, str]:
"""给代码补名称：先查自选/缓存池，池子没覆盖的用 get_stock_info 补。"""
need = [c for c in codes if c]
out: dict[str, str] = {}
for market in ("23", "50"):
for row in self.stock_pool(market):
if row["code"] in need and row["code"] not in out:
out[row["code"]] = row["name"]
todo = [c for c in need if c not in out]
for code in todo[:80]:
with _snap_lock:
info = tqs.get_stock_info(code, ["Name"]) or {}
out[code] = info.get("Name", code)
return out
# ---------- K 线：本地日线库优先，缺的再向 tqs 补 ---------- #
def last_trading_day(self) -> str:
"""最近一个交易日（用来判断本地库是否过期）。结果缓存 10 分钟。"""
key = "last_day"
cached = _cache.get(key)
if cached is not None:
return cached
from datetime import date, timedelta
start = (date.today() - timedelta(days=15)).strftime("%Y-%m-%d")
end = date.today().strftime("%Y-%m-%d")
days = tqs.get_trading_dates("SH", start, end, -1) or []
raw = str(sorted(days)[-1])[:8] if days else ""
# 统一成 YYYY-MM-DD，和本地日线库里的日期格式一致（否则比较永远不相等 → 每天重复全量取数）
out = f"{raw[:4]}-{raw[4:6]}-{raw[6:8]}" if len(raw) == 8 and raw.isdigit() else end
_cache.set(key, out, ttl=600)
return out
def kline(self, codes: list[str], count: int = 120, dividend_type: str = "front",
use_local: bool = True, max_age_min: float = 10.0) -> pd.DataFrame:
"""批量日 K：返回含 code/date/ohlcv/amount 的长表。
策略：先查本地 SQLite 日线库；只有「库里没有」或「库里的最新日期早于最近交易日」的代码才向 tqs 请求，
取回后立刻写回本地。实测 tqs 约 40ms/只，300 只全量刷新要 13 秒；本地库命中后同样的查询是毫秒级，
这就是「股软」和「一次性脚本」的分水岭——行情随时能重下，但历史必须自己存。
max_age_min 是给「今天这根」设的新鲜度上限：盘中写的当日 bar 是不完整残量，
光靠日期判断会误判成最新而永不重取，超过这个分钟数就重新回源补一次。
"""
codes = [c for c in codes if c]
if not codes:
return pd.DataFrame()
if not use_local:
return self._kline_remote(codes, count, dividend_type)
today = self.last_trading_day()
local = store.kline_read(codes, count)
have = store.kline_counts(codes)
newest: dict[str, str] = {}
if not local.empty:
newest = {str(c): str(d)[:10] for c, d in local.groupby("code")["date"].max().items()}
# 只有「完全没数据」「最新日期早于最近交易日」「本地根数不够本次要的 count」才回源
need = {c for c in codes
if have.get(c, 0) < count or newest.get(c, "") < today or c not in newest}
# 今天这根必须单独判新鲜度：盘中取的当日 bar 是不完整的残量，而
# 「库里最新日期 == 今天」会让上面的判断直接通过，于是 09:37 写进去的量能一直用到明天。
# 收盘后这根残量也不对（实测 600000.SH 库里 8,378,577 股 vs 实际全天 8,050 万股）。
# 判据用写入时刻 updated：距今超过 max_age 分钟就重取，收盘后自然会补一次终值并稳定下来。
if newest:
age = store.kline_age_minutes(codes)
now = datetime.now()
for c in list(need) + [c for c in codes if newest.get(c) == today]:
if c in newest and newest[c] == today:
wrote = age.get(c)
if wrote is not None and (now - wrote).total_seconds() / 60 >= max_age_min:
need.add(c)
if need:
need = sorted(need)
remote = self._kline_remote(need, count, dividend_type)
if not remote.empty:
store.kline_write(remote)
local = pd.concat([local, remote], ignore_index=True) if not local.empty else remote
if local.empty:
return pd.DataFrame()
local = local.drop_duplicates(subset=["code", "date"], keep="last")
local = local.sort_values(["code", "date"]).groupby("code", group_keys=False).tail(count)
return local.reset_index(drop=True)
def _kline_remote(self, codes: list[str], count: int, dividend_type: str) -> pd.DataFrame:
"""直接向 tqs 批量取日 K（不落库）。"""
key = f"kline:{dividend_type}:{count}:{','.join(codes)}"
cached = _cache.get(key)
if cached is not None:
return cached
start = _t()
raw = tqs.get_market_data(
field_list=FIELDS_DAILY,
stock_list=codes,
period="1d",
count=count,
dividend_type=dividend_type,
) or {}
_mark(start)
if not raw:
return pd.DataFrame()
frames = []
for code in codes:
close = raw.get("Close")
if close is None or code not in getattr(close, "columns", []):
continue
block = {f: raw[f][code] for f in FIELDS_DAILY if f in raw and code in raw[f]}
if not block:
continue
df = pd.DataFrame(block)
df.columns = [c.lower() for c in df.columns]
df["code"] = code
frames.append(df.reset_index().rename(columns={df.index.name or "index": "date"}))
out = pd.concat(frames, ignore_index=True) if frames else pd.DataFrame()
for col in ("open", "high", "low", "close", "volume", "amount"):
if col in out.columns:
out[col] = pd.to_numeric(out[col], errors="coerce")
out["date"] = out["date"].astype(str).str.slice(0, 10)
out = out.dropna(subset=["close"]).sort_values(["code", "date"])
_cache.set(key, out, ttl=60)
return out
# ---------- 实时快照（批量） ---------- #
SNAP_FIELDS = ["Now", "LastClose", "Open", "High", "Low", "Volume", "Amount", "Average"]
def snapshots(self, codes: list[str], chunk: int = 200) -> dict[str, dict]:
"""批量快照：走 tqs 1.1.1 的 get_market_snapshot_batch，一次 RPC 传整批代码。
实测 300 只沪深300 成分股 ≈ 0.13 秒；若用单只 get_market_snapshot 循环，300 次 ≈ 9.3 秒（慢约 70 倍）。
这是 1.1.1 新增批量接口带来的直接收益，也是「股软敢做全池实时刷新」的前提。
"""
out: dict[str, dict] = {}
codes = [c for c in codes if c]
for i in range(0, len(codes), chunk):
with _snap_lock:
batch = tqs.get_market_snapshot_batch(codes[i:i + chunk], self.SNAP_FIELDS) or {}
for code, s in batch.items():
if not isinstance(s, dict):
continue
now, last = f2v(s.get("Now")), f2v(s.get("LastClose"))
out[code] = {
"now": now,
"last_close": last,
"open": f2v(s.get("Open")),
"high": f2v(s.get("High")),
"low": f2v(s.get("Low")),
"volume": f2v(s.get("Volume")),
"amount": f2v(s.get("Amount")),
"avg": f2v(s.get("Average")),
"zaf": round((now - last) / last * 100, 2) if last else 0.0,
}
return out
# ---------- 资金 / 估值 / 涨停（扩展信息，批量） ---------- #
MORE_FIELDS = ["ZAF", "fHSL", "Zjl", "Zjl_HB", "Ltsz", "Zsz", "StaticPE_TTM", "PB_MRQ",
"ZTPrice", "DTPrice", "ZAFPre5", "ZAFPre20", "YearZTDay", "BetaValue"]
def more(self, code: str) -> dict:
"""单只扩展信息（批量接口的薄封装）。"""
return self.more_batch([code]).get(code, {})
def more_batch(self, codes: list[str], chunk: int = 300) -> dict[str, dict]:
"""批量扩展信息：主买净额、主力净流入、换手率、市值、PE/PB、涨停价、N日涨幅。
两个资金口径都要留着，别混（more_info 里现成就有）：
Zjl = 主买净额 = Σ(4 档主买) - Σ(4 档主卖)，主动性买卖之差
Zjl_HB = 主力净流入 = (特大买 + 大买) - (特大卖 + 大卖)
一次批量调用就把两个口径都拿到，不用自己再去解析 4×4 的 Amo 矩阵。
实测 300 只 ≈ 0.25 秒；单只 get_more_info 循环 300 次需 9 秒以上，批量快约 35 倍。
"""
out: dict[str, dict] = {}
codes = [c for c in codes if c]
for i in range(0, len(codes), chunk):
with _snap_lock:
batch = tqs.get_more_info_batch(codes[i:i + chunk], self.MORE_FIELDS) or {}
for code, m in batch.items():
if not isinstance(m, dict):
continue
out[code] = {
"zaf": f2v(m.get("ZAF")), # 当日涨幅 %
"hsl": f2v(m.get("fHSL")), # 换手率 %
"zjl_buy": f2v(m.get("Zjl")), # 主买净额（万元），= Σ四档Z-Σ四档P
"zjl_main": f2v(m.get("Zjl_HB")), # 主力净流入（万元），= 特大+大的 B-S
"ltsz": f2v(m.get("Ltsz")), # 流通市值（万元）
"zsz": f2v(m.get("Zsz")), # 总市值（万元）
"pe": f2v(m.get("StaticPE_TTM")), # 静态市盈率
"pb": f2v(m.get("PB_MRQ")), # 市净率
"zt_price": f2v(m.get("ZTPrice")), # 涨停价
"dt_price": f2v(m.get("DTPrice")), # 跌停价
"zaf5": f2v(m.get("ZAFPre5")), # 5 日涨幅
"zaf20": f2v(m.get("ZAFPre20")), # 20 日涨幅
"year_zt": f2v(m.get("YearZTDay")), # 年内涨停天数
"beta": f2v(m.get("BetaValue")), # Beta
}
return out
# ---------- 集合竞价（1.1.1 新接口） ---------- #
def auction(self, codes: list[str], chunk: int = 200) -> dict:
"""集合竞价时序：虚拟匹配价 / 匹配量 / 挂单量。盘中 9:15-9:25 才有过程值。"""
out: dict[str, dict] = {}
codes = [c for c in codes if c]
for i in range(0, len(codes), chunk):
with _snap_lock:
batch = tqs.get_call_auction_batch(codes[i:i + chunk]) or {}
out.update({k: v for k, v in batch.items() if isinstance(v, dict)})
return out
# ---------- 板块 ---------- #
def sectors(self) -> list[dict]:
"""全部 A 股板块（含行业/概念/地区/风格），代码形如 880201.SH。"""
key = "sector_list"
cached = _cache.get(key)
if cached is not None:
return cached
rows = tqs.get_sector_list(list_type=1) or []
out = [{"code": r.get("Code", ""), "name": r.get("Name", "")} for r in rows if r.get("Code")]
_cache.set(key, out, ttl=3600)
return out
def sector_board(self, top: int = 20) -> tuple[list[dict], list[dict]]:
"""板块涨跌幅榜：用批量快照一次拿全部 560 个板块指数的现价/昨收，算涨幅。
早期版本这里用的是「取板块日K再比两日收盘」，实测 560 只要 26 秒；改用 get_market_snapshot_batch
后同样 560 只约 0.3 秒，页面从「等半天」变成「点开就出」。
"""
sectors = self.sectors()
codes = [s["code"] for s in sectors]
if not codes:
return [], []
snaps = self.snapshots(codes, chunk=300)
name_map = {s["code"]: s["name"] for s in sectors}
rows = []
for code, q in snaps.items():
last = q.get("last_close")
# 有些风格板块指数（配股预案、业绩预升等）昨收为 0，算出来是 -100%，直接剔除
if not last or not q.get("now"):
continue
rows.append({"code": code, "name": name_map.get(code, code),
"close": round(q.get("now", 0.0), 2), "zaf": q.get("zaf", 0.0)})
rows.sort(key=lambda r: r["zaf"], reverse=True)
# 涨幅榜：降序（涨得最多在前）；跌幅榜：按涨幅升序（跌得最多在前）
return rows[:top], sorted(rows, key=lambda r: r["zaf"])[:top]
def sector_members(self, code: str, limit: int = 500) -> list[dict]:
with _snap_lock:
rows = tqs.get_stock_list_in_sector(code, 0, 1) or []
return [{"code": r.get("Code", ""), "name": r.get("Name", "")} for r in rows if r.get("Code")][:limit]
# ---------- 集合排名行情 get_domain_sorthq（1.2.2 新接口） ---------- #
# 这是「横向排序」：在某个「域」（市场/板块/指数成分）内按某一栏直接排出名次，
# 一次 RPC 出榜单，不用自己拉全池再 sort。域取值见官方文档，这里是常用几个的映射。
# （domain, 中文名）：6 全部A股 / 14 创业板 / 8 科创板 / 11002 沪深300 / 11053 中证500 /
# 11054 中证1000 / 31016 上证50 / 10000 综合板块。
SORTHQ_DOMAINS = {
"hs300": (11002, "沪深300"),
"sz50": (31016, "上证50"),
"zz500": (11053, "中证500"),
"zz1000": (11054, "中证1000"),
"cyb": (14, "创业板"),
"kcb": (8, "科创板"),
"all_a": (6, "全部A股"),
"sector": (10000, "综合板块"),
}
# 强度周期：页面上的中文名 -> get_domain_sorthq 的 colname（必须按原样写，写错直接返回空）
RPS_COLS = {"zaf": "ZAF", "zaf5": "ZAFPre5", "zaf20": "ZAFPre20", "zaf60": "ZAFPre60"}
def sorthq(self, domain: int = 6, colname: str = "ZAF", sorttype: int = 1,
wantcol: list[str] | None = None, wantnum: int = 200, startxh: int = 0) -> pd.DataFrame:
"""集合排名行情：域内按 colname 排序取一段。单页最多 200 条。"""
with _snap_lock:
df = tqs.get_domain_sorthq(domain=int(domain), colname=colname, sorttype=int(sorttype),
wantcol=list(wantcol or []), startxh=int(startxh),
wantnum=int(wantnum))
if df is None or getattr(df, "empty", True):
return pd.DataFrame()
return df
def sorthq_all(self, domain: int, colname: str = "ZAF", maxn: int = 3000) -> pd.DataFrame:
"""把域内全部证券按 colname 排好取出（分页 200 条/页，最多 maxn 条）。"""
frames, start = [], 0
while start < maxn:
page = self.sorthq(domain=domain, colname=colname, wantnum=200, startxh=start)
if page.empty:
break
frames.append(page)
if len(page) < 200: # 最后一页
break
start += 200
return pd.concat(frames, ignore_index=True) if frames else pd.DataFrame()
def rps_board(self, domain: int, col: str = "zaf20", top: int = 30) -> dict:
"""相对价格强度 RPS：域内按某周期涨幅降序，换算成 0~100 的百分位强度。
先把名次归一到 1~10000 的整数分位（10000 × (N − i) / N），再 /100 保留两位小数：
RPS = round(10000 × (N − i) / N) / 100（i 为 0 起的名次）。
这样全市场 5500+ 只股票各占一个互不相等的 RPS（相邻名次差约 0.018），不会因为
只保留一位小数而大量并列；RPS≥90 仍是「域内前 10%」，是欧奈尔 RS 评级的同款做法。
一次页取 200 条，沪深300 两页就够；返回强势榜（前 top）与弱势榜（后 top）。
"""
colname = self.RPS_COLS.get(col, "ZAFPre20")
t0 = _t()
df = self.sorthq_all(domain, colname, maxn=3000)
_mark(t0)
if df.empty:
return {"total": 0, "strong": [], "weak": []}
df = df.reset_index(drop=True)
n = len(df)
rows = []
for i, r in df.iterrows():
rows.append({
"rank": i + 1, "code": str(r.get("Code", "")), "name": str(r.get("Name", "")),
"now": f2v(r.get("Now")), "zaf": f2v(r.get("ZAF")),
"value": round(f2v(r.get(colname)), 2),
"rps": round(10000.0 * (n - i) / n) / 100.0,
})
return {"col": col, "colname": colname, "total": n,
"strong": rows[:top], "weak": rows[-top:][::-1]}
# ---------- 分时 / 分笔 ---------- #
def minute(self, code: str, date: str) -> dict:
with _snap_lock:
return tqs.get_minute_data(code, date) or {}
def ticks(self, code: str, date: str, wantnum: int = 2000) -> dict:
with _snap_lock:
return tqs.get_tick_data(code, date, 0, wantnum) or {}
def orderbook(self, code: str) -> dict:
"""五档盘口 + 内外盘 + 委增委比。
实测 tqs 1.1.1 批量快照里直接带 Buyp/Buyv/Sellp/Sellv（买五卖五），
以及 Inside/Outside（内盘/外盘）、Zangsu（委增委比）、Jjjz（竞价金额），
不需要额外的接口调用。注意这仍是五档，真正的十档委托队列要 get_pricevol，
而 1.1.1 还没实现那个接口。
"""
with _snap_lock:
s = tqs.get_market_snapshot_batch([code]) or {}
s = s.get(code) or {}
f5 = lambda v: [float(x) for x in (v or []) if x not in ("", None)]
bid_p, bid_v = f5(s.get("Buyp")), f5(s.get("Buyv"))
ask_p, ask_v = f5(s.get("Sellp")), f5(s.get("Sellv"))
n = max(len(bid_p), len(ask_p))
bid = [{"price": bid_p[i], "vol": bid_v[i] if i < len(bid_v) else 0.0}
for i in range(n) if i < len(bid_p) and bid_p[i] > 0]
ask = [{"price": ask_p[i], "vol": ask_v[i] if i < len(ask_v) else 0.0}
for i in range(n) if i < len(ask_p) and ask_p[i] > 0]
return {
"bid": bid, "ask": ask,
"now": s.get("Now"), "last_close": s.get("LastClose"),
"inside": s.get("Inside"), "outside": s.get("Outside"),
"zangsu": s.get("Zangsu"), "refresh": s.get("RefreshTime"),
}
# ---------- 交易日 ---------- #
def trading_dates(self, start: str, end: str) -> list[str]:
with _snap_lock:
return tqs.get_trading_dates("SH", start, end, -1) or []
data = MarketData()
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

297

298

299

300

301

302

303

304

305

306

307

308

309

310

311

312

313

314

315

316

317

318

319

320

321

322

323

324

325

326

327

328

329

330

331

332

333

334

335

336

337

338

339

340

341

342

343

344

345

346

347

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

369

370

371

372

373

374

375

376

377

378

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

413

414

415

416

417

418

419

420

421

422

423

424

425

426

427

428

429

430

431

432

433

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

453

454


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

297

298

299

300

301

302

303

304

305

306

307

308

309

310

311

312

313

314

315

316

317

318

319

320

321

322

323

324

325

326

327

328

329

330

331

332

333

334

335

336

337

338

339

340

341

342

343

344

345

346

347

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

369

370

371

372

373

374

375

376

377

378

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

413

414

415

416

417

418

419

420

421

422

423

424

425

426

427

428

429

430

431

432

433

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

453

454

📄

gusoft/indicators.py

MA / MACD / KDJ / RSI / BOLL / ATR / 量比 / 20 日高低

```
# -*- coding: utf-8 -*-
"""indicators.py —— 股软指标引擎：只用 pandas/numpy 手写，公式口径与通达信一致。
为什么自己写而不用 TA-Lib：
- 少装一个依赖（TA-Lib 还要编译 C 库），照抄即可跑；
- 口径可控：均线用简单均线 SMA，与通达信 MA 相同；MACD 的 DIF=EMA12-EMA26、DEA=DIF 的 EMA9。
"""
from __future__ import annotations
from datetime import datetime
import numpy as np
import pandas as pd
def ma(close: pd.Series, n: int) -> pd.Series:
return close.rolling(n, min_periods=1).mean()
def ema(close: pd.Series, n: int) -> pd.Series:
return close.ewm(span=n, adjust=False).mean()
def macd(close: pd.Series, fast: int = 12, slow: int = 26, signal: int = 9):
"""返回 (DIF, DEA, MACD柱)。柱按通达信习惯 ×2。"""
dif = ema(close, fast) - ema(close, slow)
dea = ema(dif, signal)
return dif, dea, (dif - dea) * 2
def kdj(high: pd.Series, low: pd.Series, close: pd.Series, n: int = 9, m1: int = 3, m2: int = 3):
low_n = low.rolling(n, min_periods=1).min()
high_n = high.rolling(n, min_periods=1).max()
rsv = (close - low_n) / (high_n - low_n).replace(0, np.nan) * 100
rsv = rsv.fillna(50.0)
k = rsv.ewm(com=m1 - 1, adjust=False).mean()
d = k.ewm(com=m2 - 1, adjust=False).mean()
return k, d, 3 * k - 2 * d
def rsi(close: pd.Series, n: int = 14) -> pd.Series:
diff = close.diff()
up = diff.clip(lower=0).rolling(n, min_periods=1).mean()
dn = (-diff.clip(upper=0)).rolling(n, min_periods=1).mean()
rs = up / dn.replace(0, np.nan)
return (100 - 100 / (1 + rs)).fillna(50.0)
def boll(close: pd.Series, n: int = 20, k: float = 2.0):
mid = ma(close, n)
std = close.rolling(n, min_periods=1).std().fillna(0.0)
return mid + k * std, mid, mid - k * std
def atr(high: pd.Series, low: pd.Series, close: pd.Series, n: int = 14) -> pd.Series:
pc = close.shift(1)
tr = pd.concat([high - low, (high - pc).abs(), (low - pc).abs()], axis=1).max(axis=1)
return tr.rolling(n, min_periods=1).mean()
def enrich(df: pd.DataFrame) -> pd.DataFrame:
"""给单只股票的日线表补齐指标列（输入按 date 升序的 open/high/low/close/volume）。"""
if df is None or df.empty:
return df
out = df.reset_index(drop=True).copy()
c, h, l, v = out["close"], out["high"], out["low"], out["volume"]
for n in (5, 10, 20, 60):
out[f"ma{n}"] = ma(c, n)
dif, dea, hist = macd(c)
out["dif"], out["dea"], out["macd"] = dif, dea, hist
k, d, j = kdj(h, l, c)
out["k"], out["d"], out["j"] = k, d, j
out["rsi6"] = rsi(c, 6)
out["rsi14"] = rsi(c, 14)
up, mid, dn = boll(c)
out["boll_up"], out["boll_mid"], out["boll_dn"] = up, mid, dn
out["atr14"] = atr(h, l, c)
# 量比按通达信口径：分母只取「前 5 日」均量，不含当日。
# 含当日会自我压制——当日放量越大，压得越狠（实测某日 1.73 vs 正确 2.15）。
#
# 两个容易被忽略的坑：
# 1) 停牌股的 bar 是「前一根的整根复制」（OHLCV 全同），连续停牌 7 天就有 7 根一模一样的量。
# 直接取前 5 根会全落在停牌区，基准失真（复牌票量比被压到 0.38）。
# 这里把「整根与前一根全同」的 bar 判为停牌并置空，窗口内被置空的停牌 bar 会自动跳过。
# 2) 只看 volume 相等不够——缩量到巧合相等会误伤，必须 OHLCV 四项同时相等。
same_bar = (c.eq(c.shift(1)) & v.eq(v.shift(1))
& out["high"].eq(out["high"].shift(1)) & out["low"].eq(out["low"].shift(1)))
v_eff = v.mask(same_bar)
out["suspended"] = same_bar
out["v_ma5"] = v_eff.shift(1).rolling(5, min_periods=1).mean()
out["vol_ratio"] = (v / out["v_ma5"].replace(0, np.nan)).fillna(1.0)
out["pct_chg"] = c.pct_change().fillna(0.0) * 100
out["high20"] = h.rolling(20, min_periods=1).max() # 20日最高（突破判定用）
out["low20"] = l.rolling(20, min_periods=1).min() # 20日最低
return out
def elapsed_minutes(now: datetime | None = None) -> int:
"""当日已开盘分钟数（扣掉午休 11:30-13:00），用来把量比按时间归一。
开盘前算 0，盘中按实际走过的分钟，9:30-11:30 记 120、13:00 之后继续累加。
"""
now = now or datetime.now()
m = now.hour * 60 + now.minute
if m <= 570: # 09:30 前
return 0
if m <= 690: # 09:30-11:30
return m - 570
if m < 780: # 11:30-13:00 午休
return 120
return min(120 + m - 780, 240)
def live_vol_ratio(today_vol: float, v_ma5: float, minutes: int, full_day: int = 240) -> float:
"""通达信口径的量比：(今日累计量 ÷ 已开盘分钟) ÷ (前5日均量 ÷ 240)。
收盘后 minutes 取 240，退化成「当日量 ÷ 前5日均量」，与日线口径一致。
停牌/无成交导致前5日均量为 0 时返回 1.0，避免刷出无意义的巨值。
"""
if not v_ma5 or v_ma5 <= 0 or not today_vol:
return 1.0
minutes = min(max(minutes, 1), full_day)
return (today_vol / minutes) / (v_ma5 / full_day)
def snap_volume_to_shares(snap_vol: float) -> float:
"""快照 / 分钟线的 Volume 是「手」，日线库的 volume 是「股」，两者差 100 倍。
实测对照：601238.SH 快照 69,468 手、分钟线逐根累加 69,468（完全一致），
而同日日线库记 6,947,231 股 ≈ 69,472 手 —— 只有统一换算后量比才在正确量级。
分子分母单位不一致时量比会被压到 0.01 附近，涨停票能显示成 0.19。
"""
return (snap_vol or 0.0) * 100.0
def last_snapshot(df: pd.DataFrame) -> dict:
"""把指标表压成「一行行情卡」：现价 + 均线位置 + 量比 + MACD 状态。"""
if df is None or df.empty:
return {}
r = df.iloc[-1]
prev = df.iloc[-2] if len(df) > 1 else r
def g(col, default=0.0):
v = r.get(col, default)
return float(v) if v == v else default
return {
"date": str(r["date"])[:10],
"close": round(g("close"), 3),
"pct_chg": round(g("pct_chg"), 2),
"ma5": round(g("ma5"), 3),
"ma10": round(g("ma10"), 3),
"ma20": round(g("ma20"), 3),
"ma60": round(g("ma60"), 3),
"vol_ratio": round(g("vol_ratio"), 2),
"v_ma5": round(g("v_ma5"), 0),
"rsi14": round(g("rsi14"), 2),
"k": round(g("k"), 2),
"d": round(g("d"), 2),
"dif": round(g("dif"), 4),
"dea": round(g("dea"), 4),
"above_ma20": bool(g("close") > g("ma20")),
"bull_stack": bool(g("ma5") >= g("ma10") >= g("ma20")),
"macd_gold": bool(g("dif") > g("dea") and float(prev.get("dif", 0)) <= float(prev.get("dea", 0))),
"high20": round(g("high20"), 3),
"low20": round(g("low20"), 3),
}
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

📄

gusoft/store.py

SQLite：自选、收盘快照、预警、扫描历史、本地日线库

```
# -*- coding: utf-8 -*-
"""store.py —— 股软本地库：SQLite 一个文件存自选股、每日收盘快照、预警记录、扫描历史。
为什么上 SQLite：股软最核心的资产是「历史」，行情随时能重下，但每天的收盘价、当时的预警、当时的选股结果
必须自己存。存了历史，才能做「信号回溯」——把当时的预警拿去比对后续 5 日涨跌，验证规则是否真有效。
"""
from __future__ import annotations
import json
import os
import sqlite3
import threading
from datetime import datetime
DB_PATH = os.path.join(os.path.dirname(os.path.dirname(os.path.abspath(__file__))), "data", "gusoft.db")
_lock = threading.Lock()
SCHEMA = """
CREATE TABLE IF NOT EXISTS watchlist (
code TEXT PRIMARY KEY, name TEXT, grp TEXT DEFAULT '默认', note TEXT DEFAULT '',
added_at TEXT
);
CREATE TABLE IF NOT EXISTS snapshot (
day TEXT, code TEXT, name TEXT, close REAL, pct_chg REAL, volume REAL,
hsl REAL, zjl REAL, zjl_main REAL, ltsz REAL, pe REAL, vol_ratio REAL,
zaf5 REAL, rsi14 REAL, ma5 REAL, ma20 REAL, above_ma20 INTEGER, ts TEXT,
PRIMARY KEY (day, code)
);
CREATE TABLE IF NOT EXISTS alert (
id INTEGER PRIMARY KEY AUTOINCREMENT, ts TEXT, day TEXT, code TEXT, name TEXT,
rule TEXT, level TEXT, price REAL, msg TEXT
);
CREATE TABLE IF NOT EXISTS scan_run (
id INTEGER PRIMARY KEY AUTOINCREMENT, ts TEXT, pool TEXT, rule TEXT, hits INTEGER, top_json TEXT
);
CREATE INDEX IF NOT EXISTS idx_alert_ts ON alert(ts);
CREATE INDEX IF NOT EXISTS idx_snap_day ON snapshot(day);
CREATE TABLE IF NOT EXISTS kline (
code TEXT, day TEXT, open REAL, high REAL, low REAL, close REAL,
volume REAL, amount REAL, updated TEXT,
PRIMARY KEY (code, day)
);
CREATE INDEX IF NOT EXISTS idx_kline_day ON kline(day);
"""
def _conn() -> sqlite3.Connection:
os.makedirs(os.path.dirname(DB_PATH), exist_ok=True)
con = sqlite3.connect(DB_PATH, timeout=10)
con.row_factory = sqlite3.Row
return con
def init_db() -> None:
with _lock, _conn() as con:
con.executescript(SCHEMA)
# 老库补列：CREATE TABLE IF NOT EXISTS 不会给已存在的表加字段。
# zjl 早期存的是主力净额，改口径后 zjl 存主买净额，主力净流入单独放 zjl_main。
have = {r[1] for r in con.execute("PRAGMA table_info(snapshot)")}
if "zjl_main" not in have:
con.execute("ALTER TABLE snapshot ADD COLUMN zjl_main REAL")
# 让快照导出与行情表列对齐：补 5日%、RSI14、MA5、MA20、位置。
for col, typ in (("zaf5", "REAL"), ("rsi14", "REAL"), ("ma5", "REAL"),
("ma20", "REAL"), ("above_ma20", "INTEGER")):
if col not in have:
con.execute(f"ALTER TABLE snapshot ADD COLUMN {col} {typ}")
# ----------------------------- 自选股 ----------------------------- #
def list_watch(grp: str = "") -> list[dict]:
with _lock, _conn() as con:
sql = "SELECT * FROM watchlist"
args: tuple = ()
if grp:
sql, args = sql + " WHERE grp=?", (grp,)
return [dict(r) for r in con.execute(sql + " ORDER BY added_at", args)]
def add_watch(code: str, name: str, grp: str = "默认", note: str = "") -> bool:
with _lock, _conn() as con:
try:
con.execute(
"INSERT OR REPLACE INTO watchlist(code,name,grp,note,added_at) VALUES(?,?,?,?,?)",
(code, name, grp, note, datetime.now().strftime("%Y-%m-%d %H:%M:%S")),
)
return True
except sqlite3.Error:
return False
def remove_watch(code: str) -> None:
with _lock, _conn() as con:
con.execute("DELETE FROM watchlist WHERE code=?", (code,))
def del_watch(code: str) -> None:
"""删除自选的别名（前端用 delWatch）。"""
remove_watch(code)
# ----------------------------- 收盘快照 ----------------------------- #
def save_snapshot(day: str, rows: list[dict]) -> int:
ts = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
payload = [
(day, r["code"], r.get("name", ""), r.get("close"), r.get("pct_chg"), r.get("volume"),
r.get("hsl"), r.get("zjl"), r.get("zjl_main"), r.get("ltsz"), r.get("pe"),
r.get("vol_ratio"), r.get("zaf5"), r.get("rsi14"), r.get("ma5"), r.get("ma20"),
1 if r.get("above_ma20") else 0, ts)
for r in rows
]
with _lock, _conn() as con:
con.executemany(
"INSERT OR REPLACE INTO snapshot(day,code,name,close,pct_chg,volume,hsl,zjl,zjl_main,"
"ltsz,pe,vol_ratio,zaf5,rsi14,ma5,ma20,above_ma20,ts)"
" VALUES(?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)", payload)
return len(payload)
def snapshot_days() -> list[str]:
with _lock, _conn() as con:
return [r[0] for r in con.execute("SELECT DISTINCT day FROM snapshot ORDER BY day")]
def snapshot_rows(day: str) -> list[dict]:
with _lock, _conn() as con:
return [dict(r) for r in con.execute("SELECT * FROM snapshot WHERE day=? ORDER BY pct_chg DESC", (day,))]
# ----------------------------- 预警 ----------------------------- #
def add_alert(code: str, name: str, rule: str, level: str, price: float, msg: str) -> int:
now = datetime.now()
with _lock, _conn() as con:
cur = con.execute(
"INSERT INTO alert(ts,day,code,name,rule,level,price,msg) VALUES(?,?,?,?,?,?,?,?)",
(now.strftime("%Y-%m-%d %H:%M:%S"), now.strftime("%Y-%m-%d"), code, name, rule, level, price, msg))
return int(cur.lastrowid)
def list_alerts(limit: int = 100) -> list[dict]:
with _lock, _conn() as con:
return [dict(r) for r in con.execute("SELECT * FROM alert ORDER BY id DESC LIMIT ?", (limit,))]
def clear_alerts() -> None:
with _lock, _conn() as con:
con.execute("DELETE FROM alert")
# ----------------------------- 扫描历史 ----------------------------- #
def add_scan(pool: str, rule: str, hits: int, top: list[dict]) -> int:
with _lock, _conn() as con:
cur = con.execute(
"INSERT INTO scan_run(ts,pool,rule,hits,top_json) VALUES(?,?,?,?,?)",
(datetime.now().strftime("%Y-%m-%d %H:%M:%S"), pool, rule, hits,
json.dumps(top[:20], ensure_ascii=False)))
return int(cur.lastrowid)
def list_scans(limit: int = 20) -> list[dict]:
with _lock, _conn() as con:
out = []
for r in con.execute("SELECT id,ts,pool,rule,hits FROM scan_run ORDER BY id DESC LIMIT ?", (limit,)):
out.append(dict(r))
return out
# ----------------------------- 本地日线库 ----------------------------- #
def kline_read(codes: list[str], count: int) -> "pd.DataFrame":
"""从本地日线库读每只股票最近 count 根 K 线，返回空表表示库里没有。"""
import pandas as pd
if not codes:
return pd.DataFrame()
with _lock, _conn() as con:
marks = ",".join("?" * len(codes))
rows = con.execute(
f"SELECT code, day, open, high, low, close, volume, amount FROM kline"
f" WHERE code IN ({marks}) ORDER BY code, day", codes).fetchall()
if not rows:
return pd.DataFrame()
df = pd.DataFrame(rows, columns=["code", "date", "open", "high", "low", "close", "volume", "amount"])
return df.groupby("code", group_keys=False).tail(count).reset_index(drop=True)
def kline_write(df: "pd.DataFrame") -> int:
"""写入/覆盖本地日线库。df 需含 code/date/open/high/low/close/volume(/amount)。"""
if df is None or df.empty:
return 0
payload = [
(r["code"], str(r["date"])[:10], r.get("open"), r.get("high"), r.get("low"),
r.get("close"), r.get("volume"), r.get("amount"),
datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
for _, r in df.iterrows()
]
with _lock, _conn() as con:
con.executemany(
"INSERT OR REPLACE INTO kline(code,day,open,high,low,close,volume,amount,updated)"
" VALUES(?,?,?,?,?,?,?,?,?)", payload)
return len(payload)
def kline_days() -> list[str]:
with _lock, _conn() as con:
return [r[0] for r in con.execute("SELECT DISTINCT day FROM kline ORDER BY day")]
def kline_count() -> int:
with _lock, _conn() as con:
return int(con.execute("SELECT COUNT(*) FROM kline").fetchone()[0])
def kline_counts(codes: list[str]) -> dict[str, int]:
"""每只股票在本地库里的 K 根数，用来判断「是否够本次请求的 count」。"""
out: dict[str, int] = {}
if not codes:
return out
with _lock, _conn() as con:
marks = ",".join("?" * len(codes))
for code, n in con.execute(
f"SELECT code, COUNT(*) FROM kline WHERE code IN ({marks}) GROUP BY code", codes):
out[code] = int(n)
return out
def kline_age_minutes(codes: list[str]) -> dict[str, datetime]:
"""每只股票最新一根 K 线的写入时刻，用来判断当日 bar 是不是盘中残量。
日期判断有个盲区：盘中 09:37 写入的「今天」这根，日期就已经是今天了，
「库里最新日期 == 最近交易日」会判定为最新，于是这份不完整的量能一直用下去。
只能靠写入时刻判断新鲜度。
"""
out: dict[str, datetime] = {}
if not codes:
return out
with _lock, _conn() as con:
marks = ",".join("?" * len(codes))
for code, ts in con.execute(
f"SELECT code, MAX(updated) FROM kline WHERE code IN ({marks}) GROUP BY code", codes):
try:
out[code] = datetime.strptime(str(ts)[:19], "%Y-%m-%d %H:%M:%S")
except (ValueError, TypeError):
pass
return out
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

📄

gusoft/scanner.py

六条选股规则 + 历史回溯 + 自选预警

```
# -*- coding: utf-8 -*-
"""scanner.py —— 股软的「脑子」：条件选股扫描 + 自选股预警 + 规则历史回溯。
三件事
1. screen()：给一个股票池（沪深300 / 中证500 / 全部A股 / 自选）和一条规则，扫出当天符合条件的票并排序。
2. backtest()：同一条规则放到历史每一天上跑，统计「触发后 N 日平均收益 / 胜率 / 样本数」——这是判断规则是否
真有效的唯一办法，也避免把「今天碰巧涨了」当成「信号有效」。
3. check_alerts()：对自选股跑实时预警规则，命中就写进 SQLite 预警表。
"""
from __future__ import annotations
import numpy as np
import pandas as pd
from tdxaidata import tqs
from . import data as dm
from . import indicators as ind
data = dm.data
POOLS = {
"hs300": ("沪深300", "idx:000300.SH"),
"sh50": ("上证50", "idx:000016.SH"),
"zz500": ("中证500", "idx:000905.SH"),
"zz1000": ("中证1000", "idx:000852.SH"),
"cyb": ("创业板", "m:51"),
"kcb": ("科创板", "m:52"),
"sh_main": ("上证主板", "m:7"),
"sz_main": ("深证主板", "m:8"),
"hs_a": ("沪深A股", "m:50"),
"all_a": ("所有A股", "m:5"),
}
RULES = {
"ma_bull": "均线多头排列（MA5>MA10>MA20 且价在MA20上方）",
"breakout": "放量突破20日新高（量比>1.5 且创20日新高）",
"macd_gold": "MACD金叉（DIF上穿DEA）",
"oversold": "超跌反弹（跌破MA20后当日放量收涨>2%）",
"limit_up": "涨停或准涨停（按所属板块涨停幅度）",
"main_inflow": "主力净流入（主买净额=超大单+大单买额 减 其卖额，>0）",
}
ALERT_RULES = {
"pct": ("涨跌幅超过阈值", 3.0),
"ma20_up": ("上穿20日均线", 0.0),
"ma20_dn": ("跌破20日均线", 0.0),
"vol3": ("放量3倍（量比>3）", 3.0),
"high20": ("突破20日新高", 0.0),
}
def pool_codes(pool: str) -> list[str]:
"""股票池取代码：idx: 前缀走指数成分股接口，其余走 get_stock_list。
注意别踩坑：官方文档里 market='6' 的中文名是「上证指数成份股」，实测返回 2230 只（其实是上证 A 股范围），
不是 50 只的上证 50。指数成分股要用 get_zzgz_stocklist('000016.SH')。
"""
spec = POOLS.get(pool, POOLS["hs300"])[1]
if spec.startswith("idx:"):
return tqs.get_zzgz_stocklist(index_code=spec[4:], list_type=0) or []
return [r["code"] for r in data.stock_pool(spec[2:])]
def limit_pct(code: str) -> float:
"""按代码段判断涨停幅度：创业板/科创板 20%，北交所 30%，其余 10%。"""
c = code.split(".")[0]
if c.startswith(("300", "301", "688", "689")):
return 20.0
if c.startswith(("43", "83", "87", "88", "92")):
return 30.0
return 10.0
def enrich_pool(k: pd.DataFrame) -> dict[str, pd.DataFrame]:
"""把批量 K 线拆成 {code: 指标表}。"""
out: dict[str, pd.DataFrame] = {}
if k is None or k.empty:
return out
for code, blk in k.groupby("code"):
out[code] = ind.enrich(blk.sort_values("date"))
return out
def _hit(row: pd.Series, prev: pd.Series | None, rule: str, code: str) -> bool:
c = row["close"]
if rule == "ma_bull":
return bool(row["ma5"] > row["ma10"] > row["ma20"] and c > row["ma20"])
if rule == "breakout":
return bool(c >= row["high20"] * 0.999 and row["vol_ratio"] >= 1.5 and row["pct_chg"] > 0)
if rule == "macd_gold":
return bool(row["dif"] > row["dea"] and (prev is None or prev["dif"] <= prev["dea"]))
if rule == "oversold":
return bool(prev is not None and prev["close"] < prev["ma20"]
and row["vol_ratio"] >= 1.2 and row["pct_chg"] >= 2.0)
if rule == "limit_up":
return bool(row["pct_chg"] >= limit_pct(code) - 0.6)
return False
def screen(codes: list[str], rule: str, k: pd.DataFrame, more_map: dict[str, dict] | None = None) -> list[dict]:
"""按规则扫一遍，返回命中列表（按当日涨幅排序）。"""
tables = enrich_pool(k)
hits: list[dict] = []
for code, df in tables.items():
if df.empty:
continue
row = df.iloc[-1]
prev = ind.last_snapshot(df.iloc[:-1]) if len(df) > 1 else None
if more_map is not None and rule == "main_inflow":
ok = more_map.get(code, {}).get("zjl_main", 0) > 0
else:
ok = _hit(row, prev, rule, code)
if not ok:
continue
snap = ind.last_snapshot(df)
if rule == "main_inflow":
m = more_map.get(code, {})
snap.update({"zjl": m.get("zjl_buy"), "zjl_main": m.get("zjl_main"),
"hsl": m.get("hsl"), "ltsz": m.get("ltsz")})
snap["code"] = code
snap["vol_ratio"] = round(float(row["vol_ratio"]), 2)
snap["pct_chg"] = round(float(row["pct_chg"]), 2)
hits.append(snap)
hits.sort(key=lambda r: r.get("pct_chg", 0), reverse=True)
return hits
def backtest(codes: list[str], rule: str, k: pd.DataFrame, hold: int = 5, min_period: int = 30,
more_map: dict | None = None) -> dict:
"""历史回溯：逐日判断信号，统计触发后 hold 日收益。
口径：信号在 T 日收盘后成立，T+hold 日收盘 / T 日收盘 - 1 作为持有期收益，
统计平均收益、中位数、胜率（收益>0 的比例）与样本数。这是「不含手续费与滑点」的毛收益。
关键：同时算「同期全池无条件平均收益」作为基准。行情整体下跌时，绝对收益为负不代表规则没用，
要看 excess（超额）——这才是回溯该回答的问题。
"""
tables = enrich_pool(k)
rets: list[float] = []
bench: list[float] = [] # 全池无条件基准：同期每只票每日的 hold 日收益
for code, df in tables.items():
if len(df) < min_period + hold:
continue
c = df["close"].to_numpy()
dif = df["dif"].to_numpy()
dea = df["dea"].to_numpy()
ma5 = df["ma5"].to_numpy()
ma10 = df["ma10"].to_numpy()
ma20 = df["ma20"].to_numpy()
hi = df["high"].to_numpy()
pct = df["pct_chg"].to_numpy()
vr = df["vol_ratio"].to_numpy()
zjl = np.array([(more_map or {}).get(code, {}).get("zjl_main", np.nan) for _ in range(len(df))], float)
for i in range(min_period, len(df) - hold):
fwd = c[i + hold] / c[i] - 1.0
if c[i] <= 0 or not np.isfinite(fwd):
continue
bench.append(fwd)
ok = False
if rule == "ma_bull":
ok = ma5[i] > ma10[i] > ma20[i] and c[i] > ma20[i]
elif rule == "breakout":
hi20 = hi[max(0, i - 20):i].max()
ok = c[i] >= hi20 and vr[i] >= 1.5 and pct[i] > 0
elif rule == "macd_gold":
ok = dif[i] > dea[i] and dif[i - 1] <= dea[i - 1]
elif rule == "oversold":
ok = c[i - 1] < ma20[i - 1] and vr[i] >= 1.2 and pct[i] >= 2.0
elif rule == "limit_up":
ok = pct[i] >= limit_pct(code) - 0.6
elif rule == "main_inflow":
ok = np.isfinite(zjl[i]) and zjl[i] > 0 and pct[i] > 0 and c[i] > ma20[i]
if ok:
rets.append(fwd)
b = np.array(bench) * 100
bench_avg = round(float(b.mean()), 2) if b.size else 0.0
bench_win = round(float((b > 0).mean() * 100), 1) if b.size else 0.0
if not rets:
return {"rule": rule, "hold": hold, "samples": 0, "bench_avg": bench_avg,
"bench_win_rate": bench_win, "bench_samples": int(b.size)}
arr = np.array(rets) * 100
return {
"rule": rule,
"rule_name": RULES.get(rule, rule),
"hold": hold,
"samples": int(arr.size),
"avg": round(float(arr.mean()), 2),
"median": round(float(np.median(arr)), 2),
"win_rate": round(float((arr > 0).mean() * 100), 1),
"best": round(float(arr.max()), 2),
"worst": round(float(arr.min()), 2),
"bench_avg": bench_avg,
"bench_win_rate": bench_win,
"bench_samples": int(b.size),
"excess": round(float(arr.mean()) - bench_avg, 2) or 0.0,
}
def check_alerts(frames: dict[str, pd.DataFrame], names: dict[str, str], codes: list[str],
rules: dict[str, float] | None = None) -> list[dict]:
"""自选股预警：命中规则返回预警记录 dict（由 server 落库）。"""
rules = rules or {k: v[1] for k, v in ALERT_RULES.items()}
out: list[dict] = []
for code in codes:
df = frames.get(code)
if df is None or df.empty or len(df) < 2:
continue
snap = ind.last_snapshot(df)
prev = ind.last_snapshot(df.iloc[:-1])
name = names.get(code, code)
price = snap["close"]
def add(rule_key: str, level: str, msg: str):
out.append({"code": code, "name": name, "rule": rule_key,
"level": level, "price": price, "msg": msg})
pct_th = float(rules.get("pct", 3.0))
if abs(snap["pct_chg"]) >= pct_th:
add("pct", "high", f"当日涨跌幅 {snap['pct_chg']:+.2f}%，触发阈值 {pct_th}%")
if snap["above_ma20"] and not prev.get("above_ma20"):
add("ma20_up", "mid", f"上穿20日均线 MA20={snap['ma20']}")
if (not snap["above_ma20"]) and prev.get("above_ma20"):
add("ma20_dn", "mid", f"跌破20日均线 MA20={snap['ma20']}")
if snap["vol_ratio"] >= float(rules.get("vol3", 3.0)):
add("vol3", "high", f"量比 {snap['vol_ratio']}，放量明显")
if price >= snap["high20"] * 0.999:
add("high20", "mid", f"突破20日新高 {snap['high20']}")
return out
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

📄

gusoft/server.py

标准库 http.server + JSON API（无框架）

```
# -*- coding: utf-8 -*-
"""server.py —— 股软服务端：标准库 http.server（除了 tdxaidata 不装任何东西），对外开 JSON API。
启动：
python run_gusoft.py # 默认 http://127.0.0.1:8765
接口一览
GET /api/meta 引擎信息（池子、规则、请求统计）
GET /api/quotes?pool=&top= 行情表（涨跌幅排行 / 自选）
GET /api/kline?code=&count= 单只日K + 指标（画图用）
GET /api/minute?code=&date= 分时
GET /api/sectors 板块涨幅榜
GET /api/rps?domain=&col= 强度榜 RPS（get_domain_sorthq 横向排名）
GET /api/scan?rule=&pool= 条件选股扫描
GET /api/backtest?rule=&pool= 规则历史回溯统计
GET /api/alerts 预警记录
POST /api/watch/add 加入自选（json: code,grp）
POST /api/watch/del 删除自选（json: code）
POST /api/refresh 跑一次自选股预警 + 收盘快照落库
GET /api/export?type= 导出 CSV（quotes / scan / snapshot）
"""
from __future__ import annotations
import csv
import io
import json
import threading
import time
import traceback
import urllib.parse
from datetime import date, datetime, timedelta, time as dtime
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer
from pathlib import Path
import pandas as pd
from . import data as dm
from . import indicators as ind
from . import scanner as sc
from . import store
WEB_DIR = Path(__file__).resolve().parent / "web"
_cache: dict[str, tuple[float, object]] = {}
_tls = threading.local() # 每请求一个线程，用线程局部变量做"本次请求绕过缓存"，避免污染整个进程
def _no_cache() -> bool:
return getattr(_tls, "nocache", False)
def _memo(key: str, ttl: float, fn):
"""按 key 记忆 ttl 秒。本次请求带 nocache/cold 参数时临时绕过（冒烟测试用它测真实计算耗时）。"""
if _no_cache():
return fn()
hit = _cache.get(key)
if hit and time.time() - hit[0] < ttl:
return hit[1]
val = fn()
_cache[key] = (time.time(), val)
return val
def _pool_codes(pool: str) -> list[str]:
if pool == "watch":
return [w["code"] for w in store.list_watch()]
return sc.pool_codes(pool)
def _names_for(codes: list[str]) -> dict[str, str]:
return {w["code"]: w["name"] for w in store.list_watch()} or {}
def market_state() -> str:
"""收盘态：交易日 15:30 后 / 非交易日 / 盘前 都算 closed。
收盘态下行情接口不再实时打 tqs，而是吃当天的长缓存；只有交易日盘中
（09:15~15:30）回到 open，用 15 秒短缓存做自动刷新。
"""
today = date.today().isoformat()
if dm.data.last_trading_day() != today:
return "closed"
now = datetime.now().time()
if dtime(9, 15) <= now <= dtime(15, 30):
return "open"
return "closed"
def _closed_ttl() -> float:
"""收盘态缓存时长：缓存到「下一个 09:15」，保证开盘瞬间数据立即翻新。"""
nxt = (datetime.now() + timedelta(days=1)).replace(hour=9, minute=15, second=0, microsecond=0)
return max(3600.0, (nxt - datetime.now()).total_seconds())
# --------------------------------------------------------------------------- #
# 业务处理
# --------------------------------------------------------------------------- #
def api_meta() -> dict:
codes = _pool_codes("hs300")
return {
"engine": "gusoft",
"data_api": f"tdxaidata {__import__('tdxaidata').__version__} / tqs",
# tqs 各接口的量纲并不统一，且没有任何文档标注，必须在这里写死。
# 全部实测依据（688318.SH 2026-09-29）：
# get_market_snapshot_batch: Volume=26748, Amount=20477.52
# get_market_data(1d): Volume=2674811, Amount=20477.52
# 26748 * 100 == 2674800，与日线 2674811 比值 1.00 -> 快照是「手」，日线是「股」
# 2674811 * 76.82 / 1e4 == 20534 ≈ Amount -> Amount 是「万元」不是「元」
"units": {
"quotes.volume": "手（快照口径，=日线股数÷100）",
"quotes.amount": "万元",
"quotes.zjl": "万元（more_info.Zjl，主买净额）",
"quotes.zjl_main": "万元（more_info.Zjl_HB，主力净流入）",
"kline.volume": "股（get_market_data 1d 口径）",
"minute.volume": "手（分钟线逐根累加 = 快照量）",
"vol_ratio": "倍，通达信口径 (今日累计量÷已开盘分钟)÷(前5日均量÷240)",
},
"tqs_periods": "合法值 1m 5m 10m 15m 30m 1h 1d 45d 1w 1mon 1q 1y；"
"月线写 '1mon'，写 '1M' 会被 lower() 成 '1m' 静默返回分钟线",
"day": date.today().strftime("%Y-%m-%d"),
"server_time": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
"market_state": market_state(),
"pools": {k: v[0] for k, v in sc.POOLS.items()},
"rps_domains": {k: v[1] for k, v in dm.data.SORTHQ_DOMAINS.items()},
"rps_cols": {"zaf5": "5日涨幅", "zaf20": "20日涨幅", "zaf60": "60日涨幅"},
"rules": sc.RULES,
"alert_rules": {k: v[0] for k, v in sc.ALERT_RULES.items()},
"hs300_size": len(codes),
"local_bars": store.kline_count(),
"local_days": len(store.kline_days()),
"last_trading_day": dm.data.last_trading_day(),
"calls": dm.CALL_STATS,
"watch": len(store.list_watch()),
"watchlist": store.list_watch(),
"alerts": len(store.list_alerts(limit=999)),
"snap_days": store.snapshot_days(),
}
def api_watch_add(code: str, grp: str = "默认") -> dict:
"""加自选：只认标准 6 位代码，兼容用户直接输 600519 / 600519.SH。"""
raw = (code or "").strip().upper()
code6 = raw.split(".")[0]
if len(code6) != 6 or not code6.isdigit():
return {"ok": False, "error": "代码格式应为 600519 或 600519.SH"}
full = raw if "." in raw else (code6 + (".SH" if code6[0] in "56" else ".SZ"))
name = dm.data.names([full]).get(full, full)
store.add_watch(full, name, grp or "默认")
return {"ok": True, "code": full, "name": name, "grp": grp or "默认"}
def api_watch_del(code: str) -> dict:
raw = (code or "").strip().upper()
store.del_watch(raw)
return {"ok": True, "code": raw}
def api_quotes(pool: str = "hs300", top: int = 30) -> dict:
"""行情表：全池用批量快照+扩展信息（亚秒级），只对要显示的 N 行取日K算技术指标。
这是股软页面能做到 15 秒自动刷新的关键：不为一整屏表格去拉 300 只 × 25 根 K 线（实测 13 秒），
而是先用两次批量 RPC 排好序，再只给要显示的行补指标。
"""
codes = _pool_codes(pool)
limit = int(top) if pool != "watch" else len(codes)
closed = market_state() == "closed" # 收盘后数据不会变，行情表吃长缓存，不再 15 秒打一次 tqs
def build():
t0 = time.time()
snaps = dm.data.snapshots(codes) # 批量快照：现价 / 实时涨跌幅
mores = dm.data.more_batch(codes) # 批量扩展：主力净额 / 换手 / PE / 5日20日涨幅
names = dm.data.names(codes)
order = sorted(snaps.items(), key=lambda kv: kv[1].get("zaf", 0.0), reverse=True)
shown = order[:limit] if limit else order
show_codes = [c for c, _ in shown]
k = dm.data.kline(show_codes, count=25) # 只补显示行的均线 / RSI / 量比
tables = sc.enrich_pool(k)
minutes = ind.elapsed_minutes() # 全表共用一个「已开盘分钟数」
rows = []
for code, q in shown:
m = mores.get(code, {})
row = {
"code": code, "name": names.get(code, code),
"price": q.get("now"), "close": q.get("now"),
"zaf": q.get("zaf"), "pct_chg": q.get("zaf"),
"volume": q.get("volume"), "volume_unit": "手", "amount": q.get("amount"),
"zjl": m.get("zjl_buy"), "zjl_main": m.get("zjl_main"),
"hsl": m.get("hsl"), "pe": m.get("pe"),
"ltsz": m.get("ltsz"), "zaf5": m.get("zaf5"), "zaf20": m.get("zaf20"),
"year_zt": m.get("year_zt"), "zt_price": m.get("zt_price"),
}
df = tables.get(code)
if df is not None and not df.empty:
snap = ind.last_snapshot(df)
row.update({kk: vv for kk, vv in snap.items()
if kk in ("ma5", "ma10", "ma20", "ma60", "vol_ratio", "rsi14",
"above_ma20", "bull_stack", "high20", "low20", "date")})
# 盘中把量比换成通达信口径：(今日累计量÷已开盘分钟)÷(前5日均量÷240)。
# 日线里的量比是「当日量÷前5日均量」，盘中当日量还没走完，直接用会一路偏低。
# 注意：快照/分钟线的量是「手」，v_ma5 来自日线库是「股」，必须先 ×100 换算。
v_ma5 = float(snap.get("v_ma5") or 0.0)
if v_ma5 > 0:
today_shares = ind.snap_volume_to_shares(row.get("volume") or 0.0)
row["vol_ratio"] = round(
ind.live_vol_ratio(today_shares, v_ma5, minutes), 2)
rows.append(row)
pool_zaf = [q.get("zaf", 0.0) for q in snaps.values()]
return {
"pool": pool, "n": len(snaps), "shown": len(rows),
"market_state": "closed" if closed else "open",
"up": sum(1 for v in pool_zaf if v > 0), "down": sum(1 for v in pool_zaf if v < 0),
"flat": sum(1 for v in pool_zaf if v == 0),
"limit_up": sum(1 for r in rows if r["zaf"] >= sc.limit_pct(r["code"]) - 0.6),
"net_zjl": round(sum(mores.get(c, {}).get("zjl_buy") or 0 for c in snaps), 1),
"net_zjl_main": round(sum(mores.get(c, {}).get("zjl_main") or 0 for c in snaps), 1),
"seconds": round(time.time() - t0, 2), "rows": rows,
}
return _memo(f"quotes:{pool}:{top}", _closed_ttl() if closed else 10.0, build)
def api_kline(code: str, count: int = 120) -> dict:
"""单只日K + 指标。多取 70 根「预热」K 线再截断，保证 MA60 等长周期指标在首根就有值。"""
def build():
k = dm.data.kline([code], count=int(count) + 70)
if k.empty:
return {"code": code, "bars": []}
df = ind.enrich(k).tail(int(count))
df["date"] = df["date"].astype(str).str.slice(0, 10)
cols = ["date", "open", "high", "low", "close", "volume", "ma5", "ma10", "ma20", "ma60",
"dif", "dea", "macd", "k", "d", "j", "rsi14", "boll_up", "boll_mid", "boll_dn",
"pct_chg", "vol_ratio"]
out = df[cols].round(3)
name = dm.data.names([code]).get(code, code)
return {"code": code, "name": name, "bars": out.to_dict("records")}
return _memo(f"kline:{code}:{count}", 30, build)
def api_minute(code: str, day: str = "") -> dict:
"""分时 + 五档盘口：240 根 1 分钟线 + 买五卖五与内外盘，用于日内盯盘。
day 留空时取最近交易日——tqs 的 get_minute_data 不接受空日期，会返回空数组。
"""
def build():
t0 = time.time()
d = day or dm.data.last_trading_day()
raw = dm.data.minute(code, d) or {}
name = dm.data.names([code]).get(code, code)
nums = lambda key: [float(v) for v in raw.get(key, []) if v not in ("", None)]
price, avg = nums("Price"), nums("Average")
out = {
"code": code, "name": name, "day": d,
"times": [str(t) for t in raw.get("Time", [])],
"price": price, "avg": avg, "volume": nums("Volume"),
"orderbook": dm.data.orderbook(code),
"seconds": round(time.time() - t0, 2),
}
if price:
out.update({"last": price[-1], "high": max(price), "low": min(price),
"pct": round((price[-1] / price[0] - 1) * 100, 2) if price[0] else 0.0})
return out
return _memo(f"minute:{code}:{day}", 30, build)
def api_sectors(top: int = 20) -> dict:
def build():
t0 = time.time()
up, down = dm.data.sector_board(top=int(top))
return {"up": up, "down": down, "seconds": round(time.time() - t0, 2)}
return _memo(f"sectors:{top}", 60, build)
def api_scan(rule: str, pool: str = "hs300", count: int = 120, keep: bool = True) -> dict:
codes = _pool_codes(pool)
t0 = time.time()
k = dm.data.kline(codes, count=int(count))
more_map = None
if rule == "main_inflow":
more_map = dm.data.more_batch(codes) # 批量扩展 0.3s 覆盖全池，不用再截前 120 只
hits = sc.screen(codes, rule, k, more_map)
if hits: # 给命中股补上中文名，前端和导出都要用
hit_names = dm.data.names([h["code"] for h in hits])
for h in hits:
h["name"] = hit_names.get(h["code"], h["code"])
elapsed = round(time.time() - t0, 2)
if keep and hits:
store.add_scan(pool, rule, len(hits), hits)
return {"pool": pool, "pool_name": sc.POOLS.get(pool, ("自选",))[0], "rule": rule,
"rule_name": sc.RULES.get(rule, rule), "universe": len(codes), "hits": hits,
"seconds": elapsed, "scan_time": datetime.now().strftime("%H:%M:%S")}
def api_backtest(rule: str, pool: str = "hs300", count: int = 250, hold: int = 5) -> dict:
codes = _pool_codes(pool)
t0 = time.time()
k = dm.data.kline(codes, count=int(count))
more_map = dm.data.more_batch(codes) if rule == "main_inflow" else None
stat = sc.backtest(codes, rule, k, hold=int(hold), more_map=more_map)
stat.update({"pool": pool, "pool_name": sc.POOLS.get(pool, ("自选",))[0],
"universe": len(codes), "window": f"最近{count}个交易日",
"seconds": round(time.time() - t0, 2),
"note": "毛收益，不含手续费与滑点；T日收盘信号，T+N日收盘计收益；"
"基准=同期全池无条件平均，超额才是规则的真实贡献"})
return stat
def api_rps(domain: str = "hs300", col: str = "zaf20", top: int = 30) -> dict:
"""强度榜：域内按某周期涨幅排 RPS（get_domain_sorthq 的横向排名能力）。
domain 支持 SORTHQ_DOMAINS 的 key（hs300/zz1000/all_a/sector…）或直接给数字。
"""
key = str(domain)
if key in dm.data.SORTHQ_DOMAINS:
dom_id, dom_name = dm.data.SORTHQ_DOMAINS[key]
else:
dom_id, dom_name = (int(key) if key.isdigit() else 11002), key
def build():
t0 = time.time()
board = dm.data.rps_board(dom_id, col=col, top=int(top))
board.update({"domain_key": domain, "domain_id": dom_id, "domain_name": dom_name,
"seconds": round(time.time() - t0, 2),
"day": dm.data.last_trading_day()})
return board
return _memo(f"rps:{dom_id}:{col}:{top}", 60, build)
def api_watch_list() -> dict:
return {"rows": store.list_watch()}
def api_refresh() -> dict:
"""一次「收盘巡检」：自选股跑预警 + 把全池收盘数据落库。"""
t0 = time.time()
codes = _pool_codes("watch") or _pool_codes("hs300")
k = dm.data.kline(codes, count=25)
tables = sc.enrich_pool(k)
names = dm.data.names(codes)
alerts = sc.check_alerts(tables, names, codes)
for a in alerts:
store.add_alert(a["code"], a["name"], a["rule"], a["level"], a["price"], a["msg"])
day = dm.data.last_trading_day() or date.today().strftime("%Y-%m-%d")
more_map = dm.data.more_batch(codes[:60]) # 扩展信息批量取：资金/换手/市值/PE
rows = []
for code, df in tables.items():
if df.empty:
continue
s = ind.last_snapshot(df)
m = more_map.get(code, {})
rows.append({"code": code, "name": names.get(code, code), "close": s["close"],
"pct_chg": s["pct_chg"], "volume": float(df["volume"].iloc[-1]),
"hsl": m.get("hsl"), "zjl": m.get("zjl_buy"), "zjl_main": m.get("zjl_main"),
"ltsz": m.get("ltsz"), "pe": m.get("pe"), "vol_ratio": s["vol_ratio"],
"zaf5": m.get("zaf5"), "rsi14": s.get("rsi14"), "ma5": s.get("ma5"),
"ma20": s.get("ma20"), "above_ma20": bool(s.get("above_ma20"))})
saved = store.save_snapshot(day, rows)
return {"day": day, "watch_alerts": alerts, "alerts_saved": len(alerts),
"snapshot_saved": saved, "seconds": round(time.time() - t0, 2)}
def api_export(kind: str, rule: str = "ma_bull", pool: str = "hs300") -> str:
buf = io.StringIO()
w = csv.writer(buf)
if kind == "snapshot":
rows = store.snapshot_rows(date.today().strftime("%Y-%m-%d")) or \
(store.snapshot_rows(store.snapshot_days()[-1]) if store.snapshot_days() else [])
# 列与「行情」页表格保持一致（现价/涨跌幅/主力净额/主买净额/换手/PE/5日%/量比/RSI14/MA5/MA20/位置），
# 仅在最前面加 day 作为快照的日期主键。单位后缀照旧：金额为「万」，换手/涨跌幅为「%」。
cols = ["day", "code", "name", "close", "pct_chg", "zjl_main", "zjl",
"hsl", "pe", "zaf5", "vol_ratio", "rsi14", "ma5", "ma20", "above_ma20"]
w.writerow(["day", "code", "name", "close", "pct_chg", "zjl_main_wan", "zjl_wan",
"hsl_pct", "pe", "zaf5_pct", "vol_ratio", "rsi14", "ma5", "ma20", "above_ma20"])
for r in rows:
w.writerow([r.get(k) for k in cols])
else:
res = api_scan(rule, pool, keep=False)
# 列与「选股」页结果表保持一致：代码/名称/现价/涨跌幅/量比/MA5/MA20/RSI14/DIF/DEA。
cols = ["code", "name", "close", "pct_chg", "vol_ratio", "ma5", "ma20",
"rsi14", "dif", "dea"]
w.writerow(cols)
for r in res["hits"]:
w.writerow([r.get(k) for k in cols])
return "\ufeff" + buf.getvalue()
# --------------------------------------------------------------------------- #
# HTTP
# --------------------------------------------------------------------------- #
class Handler(BaseHTTPRequestHandler):
protocol_version = "HTTP/1.1"
def log_message(self, fmt, *args): # 静默访问日志，终端只留关键输出
pass
def _send(self, body: bytes, ctype: str = "application/json; charset=utf-8", code: int = 200):
self.send_response(code)
self.send_header("Content-Type", ctype)
self.send_header("Content-Length", str(len(body)))
self.send_header("Cache-Control", "no-store")
self.end_headers()
self.wfile.write(body)
def _json(self, obj, code: int = 200):
self._send(json.dumps(obj, ensure_ascii=False, default=str).encode("utf-8"), code=code)
def do_GET(self):
u = urllib.parse.urlparse(self.path)
q = urllib.parse.parse_qs(u.query)
path = u.path
_tls.nocache = ("nocache" in q or "cold" in q) # 仅本次请求绕过记忆缓存
try:
if path in ("/", "/index.html"):
self._send((WEB_DIR / "index.html").read_bytes(), "text/html; charset=utf-8")
return
if path == "/api/meta":
return self._json(api_meta())
if path == "/api/quotes":
return self._json(api_quotes(q.get("pool", ["hs300"])[0], int(q.get("top", ["30"])[0])))
if path == "/api/kline":
return self._json(api_kline(q.get("code", ["600000.SH"])[0], int(q.get("count", ["120"])[0])))
if path == "/api/minute":
d = q.get("date", [""])[0]
return self._json(api_minute(q.get("code", ["600000.SH"])[0], d))
if path == "/api/sectors":
return self._json(api_sectors(int(q.get("top", ["20"])[0])))
if path == "/api/rps":
return self._json(api_rps(q.get("domain", ["hs300"])[0],
q.get("col", ["zaf20"])[0],
int(q.get("top", ["30"])[0])))
if path == "/api/scan":
return self._json(api_scan(q.get("rule", ["ma_bull"])[0], q.get("pool", ["hs300"])[0]))
if path == "/api/backtest":
return self._json(api_backtest(q.get("rule", ["ma_bull"])[0], q.get("pool", ["hs300"])[0],
int(q.get("count", ["250"])[0]), int(q.get("hold", ["5"])[0])))
if path == "/api/alerts":
lim = int(q.get("limit", ["60"])[0])
return self._json({"rows": store.list_alerts(lim),
"total": len(store.list_alerts(limit=9999))})
if path == "/api/watch":
return self._json(api_watch_list())
if path == "/api/export":
csv_text = api_export(q.get("type", ["snapshot"])[0],
q.get("rule", ["ma_bull"])[0],
q.get("pool", ["hs300"])[0])
self._send(csv_text.encode("utf-8"), "text/csv; charset=utf-8")
return
self._json({"error": "not found", "path": path}, 404)
except Exception as exc: # 任何异常都回 JSON，前端不会白屏
traceback.print_exc()
self._json({"error": f"{type(exc).__name__}: {exc}"}, 500)
def do_POST(self):
u = urllib.parse.urlparse(self.path)
length = int(self.headers.get("Content-Length") or 0)
raw = self.rfile.read(length) if length else b"{}"
try:
payload = json.loads(raw.decode("utf-8") or "{}")
except json.JSONDecodeError:
payload = {}
try:
if u.path == "/api/watch/add":
return self._json(api_watch_add(payload.get("code", ""), payload.get("grp", "默认")))
if u.path == "/api/watch/del":
return self._json(api_watch_del(payload.get("code", "")))
if u.path == "/api/watch": # 旧接口，保留兼容
code = (payload.get("code") or "").upper()
if not code:
return self._json({"ok": False, "msg": "缺少 code"}, 400)
name = payload.get("name") or dm.data.names([code]).get(code, code)
ok = store.add_watch(code, name, payload.get("grp", "默认"), payload.get("note", ""))
return self._json({"ok": ok, "code": code, "name": name})
if u.path == "/api/watch_del":
store.remove_watch((payload.get("code") or "").upper())
return self._json({"ok": True})
if u.path == "/api/refresh":
return self._json(api_refresh())
self._json({"error": "not found", "path": u.path}, 404)
except Exception as exc:
traceback.print_exc()
self._json({"error": f"{type(exc).__name__}: {exc}"}, 500)
def serve(host: str = "127.0.0.1", port: int = 8765):
store.init_db()
srv = ThreadingHTTPServer((host, port), Handler)
print(f"[gusoft] 数据库: {store.DB_PATH}")
print(f"[gusoft] 打开浏览器访问: http://{host}:{port}")
print("[gusoft] Ctrl+C 退出")
try:
srv.serve_forever()
except KeyboardInterrupt:
print("\n[gusoft] 已退出")
finally:
srv.server_close()
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

297

298

299

300

301

302

303

304

305

306

307

308

309

310

311

312

313

314

315

316

317

318

319

320

321

322

323

324

325

326

327

328

329

330

331

332

333

334

335

336

337

338

339

340

341

342

343

344

345

346

347

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

369

370

371

372

373

374

375

376

377

378

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

413

414

415

416

417

418

419

420

421

422

423

424

425

426

427

428

429

430

431

432

433

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

453

454

455

456

457

458

459

460

461

462

463

464

465

466

467

468

469

470

471

472

473

474

475

476

477

478

479

480

481

482

483

484

485

486

487

488

489

490

491

492

493

494

495


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

297

298

299

300

301

302

303

304

305

306

307

308

309

310

311

312

313

314

315

316

317

318

319

320

321

322

323

324

325

326

327

328

329

330

331

332

333

334

335

336

337

338

339

340

341

342

343

344

345

346

347

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

369

370

371

372

373

374

375

376

377

378

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

413

414

415

416

417

418

419

420

421

422

423

424

425

426

427

428

429

430

431

432

433

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

453

454

455

456

457

458

459

460

461

462

463

464

465

466

467

468

469

470

471

472

473

474

475

476

477

478

479

480

481

482

483

484

485

486

487

488

489

490

491

492

493

494

495

📄

gusoft/web/index.html

前端单文件，八个页签 + Canvas 手绘 K 线

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>gusoft · 本地股软（TdxAiData / tqs）</title>
<style>
* { box-sizing: border-box; }
body { margin:0; background:#0d1117; color:#e6edf3;
font-family:"Microsoft YaHei","PingFang SC",-apple-system,sans-serif; font-size:14px; }
header { display:flex; align-items:center; gap:14px; padding:10px 18px; background:#161b22;
border-bottom:1px solid #30363d; position:sticky; top:0; z-index:10; flex-wrap:wrap; }
.logo { font-size:18px; font-weight:bold; color:#ff6b3d; letter-spacing:1px; }
.logo small { color:#7d8590; font-weight:normal; font-size:12px; margin-left:6px; }
.tag { font-size:12px; color:#7d8590; border:1px solid #30363d; border-radius:10px; padding:2px 8px; }
select, button { background:#21262d; color:#e6edf3; border:1px solid #30363d; border-radius:6px;
padding:5px 10px; cursor:pointer; font-size:13px; }
button:hover { border-color:#ff6b3d; color:#ff6b3d; }
.grow { flex:1; }
nav { display:flex; gap:4px; padding:0 18px; background:#161b22; border-bottom:1px solid #30363d; }
nav button { border:none; border-radius:0; background:transparent; color:#7d8590; padding:10px 16px;
border-bottom:2px solid transparent; }
nav button.on { color:#ff6b3d; border-bottom-color:#ff6b3d; }
.panel { display:none; padding:14px 18px 30px; }
.panel.on { display:block; }
.cards { display:flex; gap:10px; flex-wrap:wrap; margin-bottom:12px; }
.card { background:#161b22; border:1px solid #30363d; border-radius:8px; padding:10px 14px; min-width:110px; }
.card b { display:block; font-size:20px; margin-top:2px; }
.lbl { color:#7d8590; font-size:12px; }
table { width:100%; border-collapse:collapse; }
th, td { padding:7px 8px; text-align:right; border-bottom:1px solid #21262d; white-space:nowrap; }
th { color:#7d8590; font-weight:normal; font-size:12px; text-align:right; position:sticky; top:0; background:#0d1117; }
th:first-child, td:first-child, th.l, td.l { text-align:left; }
tbody tr:hover { background:#1c2128; cursor:pointer; }
.up { color:#ff4d4f; } .down { color:#21c17a; } .flat { color:#8b949e; }
.mgrid { display:grid; grid-template-columns:1fr 190px; gap:10px; }
.obox { padding:8px 10px; font-size:12px; }
.orow { display:flex; justify-content:space-between; align-items:center; padding:3px 0; color:#7d8590; }
.orow.oh { border-bottom:1px solid #30363d; margin-bottom:4px; padding-bottom:5px; color:#e6edf3; }
.obk { display:flex; justify-content:space-between; font-family:monospace; padding:2.5px 0; }
.obk .q { color:#8b949e; width:22px; }
.obk .p { width:56px; }
.obk .v { width:74px; text-align:right; }
.box { background:#161b22; border:1px solid #30363d; border-radius:8px; padding:10px; }
canvas { width:100%; display:block; }
.cols { display:flex; gap:12px; flex-wrap:wrap; }
.cols > .box { flex:1; min-width:320px; }
.row { display:flex; gap:8px; align-items:center; margin-bottom:10px; flex-wrap:wrap; }
.note { color:#7d8590; font-size:12px; }
.badge { font-size:11px; border-radius:4px; padding:1px 6px; border:1px solid #30363d; color:#7d8590; }
.hi { color:#ff6b3d; } .mid { color:#e3b341; }
#toast { position:fixed; right:16px; bottom:16px; background:#21262d; border:1px solid #ff6b3d;
color:#ff6b3d; padding:8px 14px; border-radius:6px; display:none; font-size:13px; }
</style>
</head>
<body>
<header>
<div class="logo">gusoft<small>本地股软 · 数据源 TdxAiData / tqs</small></div>
<span class="tag" id="tagApi">API 加载中…</span>
<span class="tag" id="tagTime">--</span>
<span class="tag" id="tagState" style="border-color:#21c17a;color:#21c17a">--</span>
<span class="tag" id="tagWatch">自选 0</span>
<span class="tag" id="tagAlert">预警 0</span>
<div class="grow"></div>
<button onclick="doRefresh()">收盘巡检 / 落库</button>
<a class="tag" href="/api/export?type=snapshot" style="text-decoration:none;color:#7d8590">导出CSV</a>
</header>
<nav>
<button class="on" data-p="quote" onclick="tab('quote',this)">行情</button>
<button data-p="kline" onclick="tab('kline',this)">K线</button>
<button data-p="minute" onclick="tab('minute',this)">分时</button>
<button data-p="sector" onclick="tab('sector',this)">板块</button>
<button data-p="rps" onclick="tab('rps',this)">强度榜 RPS</button>
<button data-p="scan" onclick="tab('scan',this)">选股 / 回测</button>
<button data-p="alert" onclick="tab('alert',this)">预警</button>
<button data-p="watch" onclick="tab('watch',this)">自选管理</button>
</nav>
<!-- 行情 -->
<section class="panel on" id="p-quote">
<div class="row">
<span class="lbl">股票池</span>
<select id="poolSel" onchange="loadQuotes()"></select>
<span class="lbl">显示</span>
<select id="topSel" onchange="loadQuotes()">
<option value="20">前20</option><option value="30" selected>前30</option>
<option value="60">前60</option><option value="100">前100</option>
</select>
<span class="note" id="quoteNote"></span>
</div>
<div class="cards" id="quoteCards"></div>
<div class="box" style="max-height:520px;overflow:auto">
<table><thead><tr>
<th class="l">代码</th><th class="l">名称</th><th>现价</th><th>涨跌幅</th>
<th title="主力净流入 = (特大买+大买) - (特大卖+大卖)，只看特大、大两档；more_info 的 Zjl_HB">主力净额(万)</th>
<th title="主买净额 = Σ四档主买 - Σ四档主卖，主动性买卖之差，四档全算；more_info 的 Zjl。与主力净额是两个口径，别混">主买净额(万)</th>
<th>换手%</th><th>PE</th><th>5日%</th><th>量比</th><th>RSI14</th>
<th>MA5</th><th>MA20</th><th>位置</th>
</tr></thead><tbody id="quoteBody"></tbody></table>
</div>
</section>
<!-- K线 -->
<section class="panel" id="p-kline">
<div class="row">
<span class="lbl">代码</span>
<input id="kcode" value="600000.SH" style="background:#21262d;color:#e6edf3;border:1px solid #30363d;border-radius:6px;padding:5px 8px;width:110px">
<button onclick="loadKline()">加载</button>
<span class="lbl">区间</span>
<select id="kcount" onchange="loadKline()">
<option value="60">60日</option><option value="120" selected>120日</option>
<option value="250">250日</option>
</select>
<span class="grow"></span>
<span class="note" id="kname"></span>
</div>
<div class="box"><canvas id="kchart" height="620"></canvas></div>
<div class="note" id="ktip" style="margin-top:8px">移动鼠标看十字光标数值。K线为前复权，数据由 tqs 批量取数后本地计算指标。</div>
</section>
<!-- 分时 -->
<section class="panel" id="p-minute">
<div class="row">
<span class="lbl">代码</span>
<input id="mcode" value="600000.SH" style="background:#21262d;color:#e6edf3;border:1px solid #30363d;border-radius:6px;padding:5px 8px;width:110px">
<button onclick="loadMinute()">加载</button>
<span class="note" id="mnote"></span>
</div>
<div class="mgrid">
<div class="box"><canvas id="mchart" height="360"></canvas></div>
<div class="box obox">
<div class="orow oh"><span>五档盘口</span><span id="oinfo"></span></div>
<div id="obook"></div>
<div class="orow"><span class="lbl">内外盘</span><span id="oinout"></span></div>
</div>
</div>
<div class="note" style="margin-top:8px">分时数据为 tqs 1 分钟线，共 240 根；黄线为均价，白线为价格，下方为分钟成交量。</div>
</section>
<!-- 板块 -->
<section class="panel" id="p-sector">
<div class="row"><span class="note" id="sectorNote"></span>
<button onclick="loadSectors()">重新拉取</button></div>
<div class="cols">
<div class="box"><p style="margin:0 0 8px;color:#ff4d4f">▲ 板块涨幅榜</p>
<table><tbody id="secUp"></tbody></table></div>
<div class="box"><p style="margin:0 0 8px;color:#21c17a">▼ 板块跌幅榜</p>
<table><tbody id="secDn"></tbody></table></div>
</div>
</section>
<!-- 强度榜 RPS -->
<section class="panel" id="p-rps">
<div class="row">
<span class="lbl">强度域</span>
<select id="rpsDomain" onchange="loadRps()"></select>
<span class="lbl">强度周期</span>
<select id="rpsCol" onchange="loadRps()">
<option value="zaf5">5日涨幅</option>
<option value="zaf20" selected>20日涨幅</option>
<option value="zaf60">60日涨幅</option>
</select>
<button onclick="loadRps()">刷新</button>
<span class="note" id="rpsNote"></span>
</div>
<div class="note" style="margin:0 0 10px;line-height:1.9">
<b>get_domain_sorthq</b>：在指定「域」（指数成分/全部A股/板块）内按某一栏一次排好名次——横向排序的接口，
不用自己拉全池再 sort。RPS = round(10000 × (N − 名次) / N) / 100（先归一到 1~10000 再 /100，保留两位小数，
全市场 5500+ 只各占一个不等值），RPS≥90 即域内前 10%，是欧奈尔 RS 评级的同款口径。
</div>
<div class="cols">
<div class="box"><p style="margin:0 0 8px;color:#ff4d4f">▲ 个股 RPS 强势榜</p>
<table><thead><tr><th class="l">#</th><th class="l">名称</th><th>现价</th><th>涨幅</th>
<th id="rpsValHead">20日%</th><th>RPS</th></tr></thead>
<tbody id="rpsUp"></tbody></table></div>
<div class="box"><p style="margin:0 0 8px;color:#e3b341">◆ 板块 RPS 强势榜</p>
<table><thead><tr><th class="l">#</th><th class="l">板块</th><th>涨幅</th>
<th id="rpsSecValHead">20日%</th><th>RPS</th></tr></thead>
<tbody id="rpsSec"></tbody></table></div>
</div>
</section>
<!-- 选股 / 回测 -->
<section class="panel" id="p-scan">
<div class="row">
<span class="lbl">规则</span><select id="ruleSel"></select>
<span class="lbl">股票池</span><select id="scanPool"></select>
<button onclick="runScan()">开始扫描</button>
<span class="note" id="scanNote"></span>
</div>
<div class="box" style="max-height:360px;overflow:auto;margin-bottom:14px">
<table><thead><tr><th class="l">代码</th><th class="l">名称</th><th>现价</th><th>涨跌幅</th><th>量比</th>
<th>MA5</th><th>MA20</th><th>RSI14</th><th>DIF</th><th>DEA</th></tr></thead>
<tbody id="scanBody"></tbody></table>
</div>
<div class="row">
<span class="lbl">规则历史回溯（近250个交易日）</span>
<span class="lbl">持有</span>
<select id="holdSel" onchange="runBacktest()">
<option value="3">3日</option><option value="5" selected>5日</option>
<option value="10">10日</option><option value="20">20日</option>
</select>
<button onclick="runBacktest()">跑回溯</button>
<span class="note">T日收盘出信号，T+N日收盘计收益；毛收益，不含手续费与滑点</span>
</div>
<div class="cards" id="btCards"></div>
<div class="note" id="btNote" style="margin-top:8px"></div>
<div class="note" id="algoNote" style="margin-top:10px;line-height:1.9">
<b>六条规则的判定与回测口径</b><br>
ma_bull：MA5>MA10>MA20 且站上 MA20；breakout：量比≥1.5 且创 20 日新高且收涨；
macd_gold：DIF 上穿 DEA；oversold：昨破 MA20 + 量比≥1.2 + 当日涨≥2%；
limit_up：涨幅≥涨停幅−0.6%（创业板/科创板 20cm、北交所 30cm、其余 10cm）；
main_inflow：主力净额>0 且收涨且站上 MA20。<br>
回测：T 日收盘出信号，T+N 日收盘计收益（毛收益，不含手续费与滑点）；逐票逐日统计样本数、平均/中位、胜率、最好/最差，
并以「同期全池无条件平均」为基准，超额 = 规则平均 − 基准平均；默认 hold=5、min_period=30、窗口 250 交易日。
</div>
</section>
<!-- 预警 -->
<section class="panel" id="p-alert">
<div class="row">
<button onclick="doRefresh()">跑一次自选股预警</button>
<span class="note" id="alertNote"></span>
</div>
<div class="note" id="alertAlgo" style="margin:0 0 10px;line-height:1.9">
<b>5 条预警规则</b>：pct 当日涨跌幅绝对值≥3%（涨跌都触发，high）；ma20_up 收盘上穿 MA20（mid）；
ma20_dn 收盘跌破 MA20（mid）；vol3 量比≥3（high）；high20 现价≥20 日最高×0.999（mid）。
命中即带 code / name / rule / level / price 与时间戳落库到 alert 表。
</div>
<div class="box" style="max-height:520px;overflow:auto">
<table><thead><tr><th class="l">时间</th><th class="l">代码</th><th class="l">名称</th>
<th class="l">规则</th><th>价格</th><th class="l">说明</th></tr></thead>
<tbody id="alertBody"></tbody></table>
</div>
</section>
<!-- 自选管理 -->
<section class="panel" id="p-watch">
<div class="row">
<span class="lbl">代码</span>
<input id="wcode" placeholder="如 600519.SH / 000001" style="background:#21262d;color:#e6edf3;border:1px solid #30363d;border-radius:6px;padding:5px 8px;width:150px">
<button onclick="addWatch()">加入自选</button>
<span class="lbl">分组</span>
<input id="wgrp" value="默认" style="background:#21262d;color:#e6edf3;border:1px solid #30363d;border-radius:6px;padding:5px 8px;width:90px">
<span class="note" id="watchNote">自选股存在本地 SQLite（data/gusoft.db 的 watchlist 表），关掉服务也不会丢。</span>
</div>
<div class="box" style="max-height:420px;overflow:auto">
<table><thead><tr><th class="l">代码</th><th class="l">名称</th><th class="l">分组</th>
<th class="l">加入时间</th><th>操作</th></tr></thead>
<tbody id="watchBody"></tbody></table>
</div>
</section>
<div id="toast"></div>
<script>
const $ = id => document.getElementById(id);
const fmt = (v, n=2) => (v===null||v===undefined||isNaN(v)) ? '--' : Number(v).toFixed(n);
const cls = v => v>0 ? 'up' : (v<0 ? 'down' : 'flat');
function toast(msg){ const t=$('toast'); t.textContent=msg; t.style.display='block';
setTimeout(()=>t.style.display='none', 2600); }
async function jget(u){ const r = await fetch(u); return r.json(); }
async function jpost(u, body){ const r = await fetch(u, {method:'POST',
headers:{'Content-Type':'application/json'}, body: JSON.stringify(body||{})}); return r.json(); }
function tab(name, btn){
document.querySelectorAll('.panel').forEach(p=>p.classList.remove('on'));
document.querySelectorAll('nav button').forEach(b=>b.classList.remove('on'));
$('p-'+name).classList.add('on'); btn.classList.add('on');
if(name==='sector' && !$('secUp').children.length) loadSectors();
if(name==='rps' && !$('rpsUp').children.length) loadRps();
if(name==='scan' && !$('scanBody').children.length) runScan();
if(name==='kline' && !KDATA) loadKline();
if(name==='minute' && !MDATA) loadMinute();
if(name==='alert') loadAlerts();
if(name==='watch') loadWatch();
}
async function loadMeta(){
const m = await jget('/api/meta');
$('tagApi').textContent = m.data_api;
$('tagTime').textContent = m.server_time + ' · 沪深300 ' + m.hs300_size + ' 只';
const st = $('tagState');
if(m.market_state === 'open'){
st.textContent = '盘中';
st.style.borderColor = '#21c17a'; st.style.color = '#21c17a';
}else{
st.textContent = '已收盘 · 数据定格';
st.style.borderColor = '#7d8590'; st.style.color = '#7d8590';
}
$('tagWatch').textContent = '自选 ' + m.watch;
$('tagAlert').textContent = '预警 ' + m.alerts;
const opts = Object.entries(m.pools).map(([k,v])=>`<option value="${k}">${v}</option>`).join('');
$('poolSel').innerHTML = opts + '<option value="watch">我的自选</option>';
$('scanPool').innerHTML = opts;
$('ruleSel').innerHTML = Object.entries(m.rules).map(([k,v])=>`<option value="${k}">${k}｜${v}</option>`).join('');
const doms = m.rps_domains || {};
$('rpsDomain').innerHTML = Object.entries(doms)
.filter(([k])=>k!=='sector').map(([k,v])=>`<option value="${k}"${k==='hs300'?' selected':''}>${v}</option>`).join('');
MARKET_CLOSED = m.market_state !== 'open';
}
async function loadQuotes(){
const pool = $('poolSel').value, top = $('topSel').value;
$('quoteNote').textContent = '取数中…';
const d = await jget(`/api/quotes?pool=${pool}&top=${top}`);
const frozen = d.market_state === 'closed' ? ' · 已收盘，数据定格' : '';
$('quoteNote').textContent = `共 ${d.n} 只 · 批量取数 ${d.seconds}s · 15秒自动刷新${frozen}`;
$('quoteCards').innerHTML = [
['样本数', d.n, ''], ['上涨', d.up, 'up'], ['下跌', d.down, 'down'],
['平盘', d.flat, 'flat'], ['涨停/准涨停', d.limit_up, 'up'],
['主力净额合计(万)', d.net_zjl_main, cls(d.net_zjl_main)],
['主买净额合计(万)', d.net_zjl, cls(d.net_zjl)],
].map(([l,v,c])=>`<div class="card"><span class="lbl">${l}</span><b class="${c}">${v}</b></div>`).join('');
$('quoteBody').innerHTML = (d.rows||[]).map(r=>`
<tr onclick="goKline('${r.code}')">
<td class="l">${r.code}</td><td class="l">${r.name}</td>
<td>${fmt(r.price)}</td><td class="${cls(r.zaf)}">${r.zaf>0?'+':''}${fmt(r.zaf)}%</td>
<td class="${cls(r.zjl_main)}">${r.zjl_main>0?'+':''}${fmt(r.zjl_main,0)}</td>
<td class="${cls(r.zjl)}">${r.zjl>0?'+':''}${fmt(r.zjl,0)}</td>
<td>${fmt(r.hsl)}</td><td>${fmt(r.pe)}</td>
<td class="${cls(r.zaf5)}">${r.zaf5>0?'+':''}${fmt(r.zaf5,1)}%</td>
<td>${fmt(r.vol_ratio)}</td><td>${fmt(r.rsi14,1)}</td>
<td>${fmt(r.ma5)}</td><td>${fmt(r.ma20)}</td>
<td class="${r.above_ma20?'up':'down'}">${r.above_ma20?'MA20上':'MA20下'}</td>
</tr>`).join('') || '<tr><td colspan="13" class="note">该股票池暂无数据，先在自选里加几只。</td></tr>';
}
function goKline(code){
$('kcode').value = code; tab('kline', document.querySelector('nav button[data-p=kline]')); loadKline();
$('mcode').value = code;
}
// ---------- 自选管理 ----------
async function loadWatch(){
const d = await jget('/api/meta');
$('watchBody').innerHTML = (d.watchlist||[]).map(w=>`
<tr><td class="l">${w.code}</td><td class="l">${w.name||'--'}</td><td class="l">${w.grp||'默认'}</td>
<td class="l">${(w.added_at||'').slice(0,16)}</td>
<td><button onclick="delWatch('${w.code}')">删除</button>
<button onclick="goKline('${w.code}')">看K线</button></td></tr>`).join('')
|| '<tr><td colspan="5" class="note">还没有自选股。上面输入代码即可加入。</td></tr>';
$('watchNote').textContent = `当前 ${(d.watchlist||[]).length} 只，存于本地 SQLite（data/gusoft.db 的 watchlist 表），关掉服务也不会丢。`;
}
async function addWatch(){
const code = $('wcode').value.trim().toUpperCase();
if(!code) return toast('请先填代码');
const d = await jpost('/api/watch/add', {code, grp: $('wgrp').value.trim() || '默认'});
toast(d.ok ? `已加入自选：${d.name}（${d.code}）` : ('加入失败：' + (d.error||'未知原因')));
if(d.ok){ $('wcode').value=''; loadMeta(); loadWatch(); }
}
async function delWatch(code){
const d = await jpost('/api/watch/del', {code});
toast(d.ok ? `已删除 ${code}` : '删除失败');
loadMeta(); loadWatch();
}
let MDATA = null;
async function loadMinute(){
const code = $('mcode').value.trim().toUpperCase();
$('mnote').textContent = '取数中…';
const d = await jget(`/api/minute?code=${code}`);
MDATA = d;
$('mnote').innerHTML = `${d.name} ${d.code} · ${d.price.length} 根分钟线 · 现价 <b>${fmt(d.last)}</b> `
+ `<span class="${cls(d.pct)}">${d.pct>0?'+':''}${fmt(d.pct)}%</span> · 最高 ${fmt(d.high)} 最低 ${fmt(d.low)} · ${d.seconds}s`;
drawMinute(d);
drawBook(d.orderbook || {});
}
function drawBook(ob){
const clsOf = p => p > (ob.last_close||0) ? 'up' : p < (ob.last_close||0) ? 'down' : 'flat';
const row = (label, x) => `<div class="obk"><span class="q">${label}</span>`
+ `<span class="p ${clsOf(x.price)}">${fmt(x.price)}</span>`
+ `<span class="v">${fmt(x.vol,0)}</span></div>`;
let h = '';
[...(ob.ask||[])].reverse().forEach((x,i)=>{ h += row(`卖${(ob.ask.length-i)}`, x); });
h += `<div class="obk" style="border-top:1px solid #30363d;border-bottom:1px solid #30363d;margin:3px 0">`
+ `<span class="q">现</span><span class="p">${fmt(ob.now)}</span>`
+ `<span class="v">昨收 ${fmt(ob.last_close)}</span></div>`;
(ob.bid||[]).forEach((x,i)=>{ h += row(`买${i+1}`, x); });
$('obook').innerHTML = h;
$('oinfo').textContent = ob.refresh ? `刷新 ${String(ob.refresh).padStart(6,'0')}` : '';
const io = (ob.inside||0) + (ob.outside||0);
$('oinout').innerHTML = io
? `内盘 <b class="down">${fmt(ob.inside,0)}</b> / 外盘 <b class="up">${fmt(ob.outside,0)}</b>`
+ `<br>委增委比 <b>${fmt(ob.zangsu,2)}</b>`
: '';
}
function drawMinute(d){
const cv = $('mchart'), dpr = window.devicePixelRatio || 1;
const cssH = 360, cssW = cv.parentElement.clientWidth;
cv.width = cssW*dpr; cv.height = cssH*dpr; cv.style.height = cssH+'px';
const g = cv.getContext('2d'); g.setTransform(dpr,0,0,dpr,0,0);
g.clearRect(0,0,cssW,cssH);
if(!d || !d.price.length) return;
const padL=56, padR=56, padT=10, hVol=70, gap=12;
const W = cssW-padL-padR, hPrice = cssH-padT-hVol-gap;
const all = d.price.concat(d.avg);
const mx = Math.max(...all), mn = Math.min(...all);
const span = Math.max(mx-mn, mx*0.01), top = mx+span*0.15, bot = mn-span*0.15;
const X = i => padL + (W*i/(d.price.length-1));
const Y = v => padT + (top-v)/(top-bot)*hPrice;
g.strokeStyle='#21262d'; g.fillStyle='#7d8590'; g.font='11px monospace'; g.textAlign='center';
for(let i=0;i<=4;i++){
const y = padT + hPrice*i/4, val = top-(top-bot)*i/4;
g.beginPath(); g.moveTo(padL,y); g.lineTo(padL+W,y); g.stroke();
g.textAlign='right'; g.fillText(val.toFixed(2), padL-6, y+4);
g.textAlign='left'; g.fillText(((val/d.price[0]-1)*100).toFixed(1)+'%', padL+W+6, y+4);
}
for(const t of ['09:30','10:30','11:30/13:00','14:00','15:00']){
const idx = ({'09:30':0,'10:30':60,'11:30/13:00':120,'14:00':180,'15:00':239})[t];
g.textAlign='center'; g.fillText(t, X(idx), padT+hPrice+16);
}
// 昨收基准线
const y0 = Y(d.price[0]);
g.setLineDash([4,4]); g.strokeStyle='#e3b341'; g.beginPath(); g.moveTo(padL,y0); g.lineTo(padL+W,y0); g.stroke(); g.setLineDash([]);
const line = (arr,color)=>{
g.strokeStyle=color; g.lineWidth=1.4; g.beginPath();
arr.forEach((v,i)=> i? g.lineTo(X(i),Y(v)) : g.moveTo(X(i),Y(v)));
g.stroke();
};
line(d.avg,'#e3b341'); line(d.price,'#e6edf3');
// 成交量
const yTop = padT+hPrice+gap, vmax = Math.max(...d.volume, 1);
d.volume.forEach((v,i)=>{
g.fillStyle = i===0 ? '#484f58'
: (d.price[i]>=d.price[i-1] ? '#ff4d4f' : '#21c17a');
const h = (v/vmax)*(hVol-8);
g.fillRect(X(i)-1.2, yTop+hVol-h, 2.4, h);
});
g.strokeStyle='#21262d'; g.beginPath(); g.moveTo(padL,yTop); g.lineTo(padL+W,yTop); g.stroke();
}
let KDATA = null;
async function loadKline(){
const code = $('kcode').value.trim().toUpperCase(), n = $('kcount').value;
$('kname').textContent = '取数中…';
const d = await jget(`/api/kline?code=${code}&count=${n}`);
KDATA = d; $('kname').textContent = `${d.name||''} ${code} · ${d.bars.length} 根日K（前复权）`;
draw(d.bars);
}
function draw(bars){
const cv = $('kchart'), dpr = window.devicePixelRatio || 1;
const cssH = 620, cssW = cv.parentElement.clientWidth;
cv.width = cssW*dpr; cv.height = cssH*dpr; cv.style.height = cssH+'px';
const g = cv.getContext('2d'); g.setTransform(dpr,0,0,dpr,0,0);
g.clearRect(0,0,cssW,cssH);
if(!bars || !bars.length) return;
const padL=58, padR=58, padT=12;
const hPrice=330, hVol=100, hMacd=110, gap=14;
const W = cssW-padL-padR;
const closes = bars.map(b=>b.close);
let lo=Math.min(...closes), hi=Math.max(...closes);
for(const b of bars){ lo=Math.min(lo,b.low); hi=Math.max(hi,b.high); }
const pad=(hi-lo)*0.06||1; lo-=pad; hi+=pad;
const x = i => padL + W*(i+0.5)/bars.length;
const y = p => padT + (hi-p)/(hi-lo)*hPrice;
// 网格与价格轴
g.strokeStyle='#21262d'; g.fillStyle='#7d8590'; g.font='11px monospace'; g.lineWidth=1;
for(let i=0;i<=4;i++){
const p=lo+(hi-lo)*i/4, yy=y(p);
g.beginPath(); g.moveTo(padL,yy); g.lineTo(cssW-padR,yy); g.stroke();
g.fillText(p.toFixed(2), 6, yy+4);
}
// K线
const bw = Math.max(1.5, W/bars.length*0.62);
bars.forEach((b,i)=>{
const c = b.close>=b.open ? '#ff4d4f' : '#21c17a';
g.strokeStyle=c; g.fillStyle=c; g.lineWidth=1;
g.beginPath(); g.moveTo(x(i), y(b.high)); g.lineTo(x(i), y(b.low)); g.stroke();
const y1=y(Math.max(b.open,b.close)), y2=y(Math.min(b.open,b.close));
g.fillRect(x(i)-bw/2, y1, bw, Math.max(1, y2-y1));
});
// 均线
const lines=[['ma5','#f0b90b'],['ma10','#4aa3ff'],['ma20','#c77dff'],['ma60','#8b949e']];
lines.forEach(([f,col])=>{
g.strokeStyle=col; g.beginPath(); let started=false;
bars.forEach((b,i)=>{ const v=b[f]; if(v===null||v===undefined) return;
if(!started){ g.moveTo(x(i),y(v)); started=true; } else g.lineTo(x(i),y(v)); });
g.stroke();
g.fillStyle=col; g.fillText(f.toUpperCase(), cssW-padR+6, padT+ [10,24,38,52][lines.findIndex(l=>l[0]===f)]);
});
// 成交量
const yv0 = padT+hPrice+gap, vmax=Math.max(...bars.map(b=>b.volume));
g.fillStyle='#7d8590'; g.fillText('VOL', 6, yv0+12);
bars.forEach((b,i)=>{
g.fillStyle = b.close>=b.open ? 'rgba(255,77,79,.65)' : 'rgba(33,193,122,.65)';
const hh = b.volume/vmax*hVol; g.fillRect(x(i)-bw/2, yv0+hVol-hh, bw, hh);
});
// MACD
const ym0 = yv0+hVol+gap, mm=Math.max(...bars.map(b=>Math.abs(b.macd||0)))*1.2||1;
const ym = v => ym0 + hMacd/2 - v/mm*(hMacd/2);
g.strokeStyle='#30363d'; g.beginPath(); g.moveTo(padL, ym(0)); g.lineTo(cssW-padR, ym(0)); g.stroke();
g.fillStyle='#7d8590'; g.fillText('MACD', 6, ym0+12);
bars.forEach((b,i)=>{ g.fillStyle=(b.macd>=0?'#ff4d4f':'#21c17a');
const hgt=Math.abs(b.macd)/mm*(hMacd/2);
g.fillRect(x(i)-bw/2, b.macd>=0? ym(b.macd):ym(0), bw, hgt); });
[[ 'dif','#f0b90b'],['dea','#4aa3ff']].forEach(([f,col])=>{
g.strokeStyle=col; g.beginPath(); bars.forEach((b,i)=>{ const v=b[f];
if(v===null||v===undefined) return; i?g.lineTo(x(i),ym(v)):g.moveTo(x(i),ym(v)); }); g.stroke(); });
// 日期轴
g.fillStyle='#7d8590';
const step=Math.max(1, Math.floor(bars.length/6));
bars.forEach((b,i)=>{ if(i%step===0) g.fillText(b.date.slice(2), x(i)-16, cssH-6); });
// 十字光标
cv.onmousemove = ev => {
const rect=cv.getBoundingClientRect(), mx=ev.clientX-rect.left;
const i=Math.round((mx-padL)/W*bars.length-0.5);
if(i<0||i>=bars.length){ g.setTransform(dpr,0,0,dpr,0,0); draw(bars); return; }
draw(bars);
g.setTransform(dpr,0,0,dpr,0,0);
g.strokeStyle='#8b949e'; g.setLineDash([4,4]);
g.beginPath(); g.moveTo(x(i),padT); g.lineTo(x(i),ym0+hMacd); g.stroke();
g.beginPath(); g.moveTo(padL,y(bars[i].close)); g.lineTo(cssW-padR,y(bars[i].close)); g.stroke();
g.setLineDash([]);
const t=`${bars[i].date.slice(2)} 开${bars[i].open} 高${bars[i].high} 低${bars[i].low} 收${bars[i].close} `+
`涨跌${bars[i].pct_chg}% 量比${fmt(bars[i].vol_ratio)} MA5 ${fmt(bars[i].ma5)} MA20 ${fmt(bars[i].ma20)} MACD ${fmt(bars[i].macd,3)}`;
$('ktip').textContent = t;
};
cv.onmouseleave = ()=>{ $('ktip').textContent='移动鼠标看十字光标数值。K线为前复权，数据由 tqs 批量取数后本地计算指标。'; draw(bars); };
}
async function loadSectors(){
$('sectorNote').textContent='批量取 560 个板块指数的日K计算涨幅中…';
const d = await jget('/api/sectors?top=18');
$('sectorNote').textContent = `板块指数批量取数耗时 ${d.seconds}s（一次请求 560 只）`;
const row = r=>`<tr onclick="goKline('${r.code}')"><td class="l">${r.name}</td><td>${r.close}</td>
<td class="${cls(r.zaf)}">${r.zaf>0?'+':''}${r.zaf}%</td></tr>`;
$('secUp').innerHTML = d.up.map(row).join('');
$('secDn').innerHTML = d.down.map(row).join('');
}
async function loadRps(){
const dom = $('rpsDomain').value, col = $('rpsCol').value;
const colName = {zaf5:'5日', zaf20:'20日', zaf60:'60日'}[col] || '20日';
$('rpsNote').textContent = '横向排名中（get_domain_sorthq）…';
const [d, s] = await Promise.all([
jget(`/api/rps?domain=${dom}&col=${col}&top=25`),
jget(`/api/rps?domain=sector&col=${col}&top=12`),
]);
$('rpsValHead').textContent = colName + '%';
$('rpsSecValHead').textContent = colName + '%';
$('rpsNote').innerHTML = `${d.domain_name} 域内 <b>${d.total}</b> 只 · 按${colName}涨幅排名`
+ ` · get_domain_sorthq 分页一次排好 · ${d.seconds}s · ${d.day}`;
const bar = rps => `<span style="display:inline-block;width:44px;text-align:right"
class="${rps>=90?'up':(rps>=70?'mid':'flat')}">${fmt(rps,2)}</span>`;
$('rpsUp').innerHTML = (d.strong||[]).map(r=>`<tr onclick="goKline('${r.code}')">
<td class="l note">${r.rank}</td>
<td class="l">${r.name}<span class="note"> ${r.code.slice(0,6)}</span></td>
<td>${fmt(r.now)}</td>
<td class="${cls(r.zaf)}">${r.zaf>0?'+':''}${fmt(r.zaf)}%</td>
<td class="${cls(r.value)}">${r.value>0?'+':''}${fmt(r.value,1)}%</td>
<td>${bar(r.rps)}</td></tr>`).join('')
|| '<tr><td colspan="6" class="note">该域暂无数据</td></tr>';
$('rpsSec').innerHTML = (s.strong||[]).map(r=>`<tr>
<td class="l note">${r.rank}</td><td class="l">${r.name}</td>
<td class="${cls(r.zaf)}">${r.zaf>0?'+':''}${fmt(r.zaf)}%</td>
<td class="${cls(r.value)}">${r.value>0?'+':''}${fmt(r.value,1)}%</td>
<td>${bar(r.rps)}</td></tr>`).join('')
|| '<tr><td colspan="5" class="note">暂无板块数据</td></tr>';
}
async function runScan(){
const rule=$('ruleSel').value, pool=$('scanPool').value;
$('scanNote').textContent='扫描中（批量取K线 + 本地算指标）…';
const d = await jget(`/api/scan?rule=${rule}&pool=${pool}`);
$('scanNote').textContent = `${d.pool_name} ${d.universe} 只 / 命中 ${d.hits.length} 只 / 耗时 ${d.seconds}s / ${d.scan_time}`;
$('scanBody').innerHTML = d.hits.map(r=>`<tr onclick="goKline('${r.code}')">
<td class="l">${r.code}</td><td class="l">${r.name||''}</td><td>${fmt(r.close)}</td>
<td class="${cls(r.pct_chg)}">${r.pct_chg>0?'+':''}${fmt(r.pct_chg)}%</td>
<td>${fmt(r.vol_ratio)}</td><td>${fmt(r.ma5)}</td><td>${fmt(r.ma20)}</td>
<td>${fmt(r.rsi14,1)}</td><td>${fmt(r.dif,3)}</td><td>${fmt(r.dea,3)}</td></tr>`).join('')
|| '<tr><td colspan="10" class="note">今日无票满足该条件</td></tr>';
}
async function runBacktest(){
const rule=$('ruleSel').value, pool=$('scanPool').value, hold=$('holdSel').value;
$('btCards').innerHTML = '<div class="card"><span class="lbl">回溯中</span><b>…</b></div>';
const d = await jget(`/api/backtest?rule=${rule}&pool=${pool}&count=250&hold=${hold}`);
if(!d.samples){ $('btCards').innerHTML='<div class="card"><span class="lbl">样本不足</span><b>--</b></div>'; return; }
$('btCards').innerHTML = [
['样本数', d.samples, ''], ['平均收益(%)', fmt(d.avg), cls(d.avg)],
['超额(%)', fmt(d.excess), cls(d.excess)],
['同期基准(%)', fmt(d.bench_avg), cls(d.bench_avg)],
['中位数(%)', fmt(d.median), cls(d.median)], ['胜率(%)', fmt(d.win_rate,1), ''],
['基准胜率(%)', fmt(d.bench_win_rate,1), ''],
['最好(%)', fmt(d.best), 'up'], ['最差(%)', fmt(d.worst), 'down'],
['持有(交易日)', d.hold, ''], ['耗时(s)', d.seconds, ''],
].map(([l,v,c])=>`<div class="card"><span class="lbl">${l}</span><b class="${c}">${v}</b></div>`).join('');
$('btNote').textContent = d.note;
}
async function loadAlerts(){
const d = await jget('/api/alerts?limit=80');
$('alertNote').textContent = `本地已存 ${d.rows.length} 条（库里共 ${(d.total ?? d.rows.length)} 条），来源：自选股每日收盘巡检`;
$('alertBody').innerHTML = d.rows.map(r=>`<tr>
<td class="l note">${r.ts}</td><td class="l">${r.code}</td><td class="l">${r.name}</td>
<td class="l ${r.level}">${r.rule}</td><td>${fmt(r.price)}</td><td class="l note">${r.msg}</td>
</tr>`).join('') || '<tr><td colspan="6" class="note">还没有预警记录，点上方「跑一次自选股预警」或先加自选股。</td></tr>';
}
async function doRefresh(){
toast('巡检中…');
const d = await jpost('/api/refresh', {});
toast(`巡检完成：预警 ${d.alerts_saved} 条，快照落库 ${d.snapshot_saved} 行，耗时 ${d.seconds}s`);
loadMeta(); loadAlerts(); loadQuotes();
}
let MARKET_CLOSED = false;
loadMeta().then(loadQuotes);
setInterval(()=>{ if(!MARKET_CLOSED && $('p-quote').classList.contains('on')) loadQuotes(); }, 15000);
window.addEventListener('resize', ()=>{
if(KDATA) draw(KDATA.bars);
if(MDATA) drawMinute(MDATA);
});
</script>
</body>
</html>
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

297

298

299

300

301

302

303

304

305

306

307

308

309

310

311

312

313

314

315

316

317

318

319

320

321

322

323

324

325

326

327

328

329

330

331

332

333

334

335

336

337

338

339

340

341

342

343

344

345

346

347

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

369

370

371

372

373

374

375

376

377

378

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

413

414

415

416

417

418

419

420

421

422

423

424

425

426

427

428

429

430

431

432

433

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

453

454

455

456

457

458

459

460

461

462

463

464

465

466

467

468

469

470

471

472

473

474

475

476

477

478

479

480

481

482

483

484

485

486

487

488

489

490

491

492

493

494

495

496

497

498

499

500

501

502

503

504

505

506

507

508

509

510

511

512

513

514

515

516

517

518

519

520

521

522

523

524

525

526

527

528

529

530

531

532

533

534

535

536

537

538

539

540

541

542

543

544

545

546

547

548

549

550

551

552

553

554

555

556

557

558

559

560

561

562

563

564

565

566

567

568

569

570

571

572

573

574

575

576

577

578

579

580

581

582

583

584

585

586

587

588

589

590

591

592

593

594

595

596

597

598

599

600

601

602

603

604

605

606

607

608

609

610

611

612

613

614

615

616

617

618

619

620

621

622

623

624

625

626

627

628

629


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

297

298

299

300

301

302

303

304

305

306

307

308

309

310

311

312

313

314

315

316

317

318

319

320

321

322

323

324

325

326

327

328

329

330

331

332

333

334

335

336

337

338

339

340

341

342

343

344

345

346

347

348

349

350

351

352

353

354

355

356

357

358

359

360

361

362

363

364

365

366

367

368

369

370

371

372

373

374

375

376

377

378

379

380

381

382

383

384

385

386

387

388

389

390

391

392

393

394

395

396

397

398

399

400

401

402

403

404

405

406

407

408

409

410

411

412

413

414

415

416

417

418

419

420

421

422

423

424

425

426

427

428

429

430

431

432

433

434

435

436

437

438

439

440

441

442

443

444

445

446

447

448

449

450

451

452

453

454

455

456

457

458

459

460

461

462

463

464

465

466

467

468

469

470

471

472

473

474

475

476

477

478

479

480

481

482

483

484

485

486

487

488

489

490

491

492

493

494

495

496

497

498

499

500

501

502

503

504

505

506

507

508

509

510

511

512

513

514

515

516

517

518

519

520

521

522

523

524

525

526

527

528

529

530

531

532

533

534

535

536

537

538

539

540

541

542

543

544

545

546

547

548

549

550

551

552

553

554

555

556

557

558

559

560

561

562

563

564

565

566

567

568

569

570

571

572

573

574

575

576

577

578

579

580

581

582

583

584

585

586

587

588

589

590

591

592

593

594

595

596

597

598

599

600

601

602

603

604

605

606

607

608

609

610

611

612

613

614

615

616

617

618

619

620

621

622

623

624

625

626

627

628

629
