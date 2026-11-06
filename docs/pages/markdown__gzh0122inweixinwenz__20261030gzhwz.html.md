---
title: "通达信TdxAiData：三行代码拿到 F10 综合信息 (opens new window)"
source_url: https://help.tdx.com.cn/quant/docs/markdown/gzh0122inweixinwenz/20261030gzhwz.html
fetched_at: 2026-11-06T06:04:01Z
---

# # 通达信TdxAiData：三行代码拿到 F10 综合信息 (opens new window)

通达信TdxAiData：Python独立数据接口，无需启动客户端 (opens new window)

**此功能为付费功能，详细使用过程请查看这篇公众号文章**

📄

server.py

取和处理数据

```
# -*- coding: utf-8 -*-
"""
通达信 F10 综合信息 · 本地服务
------------------------------------------------
接口：tqs.get_misc_data(stock_code) —— TdxAiData
启动：python server.py → http://127.0.0.1:8791
可选环境变量：F10_PORT（默认 8791）、F10_PY（无意义，仅记录）
"""
import json
import os
import re
import sys
import threading
import time
from http.server import HTTPServer, ThreadingHTTPServer, BaseHTTPRequestHandler
from urllib.parse import urlparse, parse_qs
BASE = os.path.dirname(os.path.abspath(__file__))
PORT = int(os.environ.get("F10_PORT", "8791"))
CACHE_TTL = 30
_cache = {}
_lock = threading.Lock()
# ---------------------------------------------------------------- 代码规范化
def normalize(raw: str):
"""把用户输入统一成 688318.SH 这种带后缀形式，非法返回 None"""
s = (raw or "").strip().upper().replace(" ", "").replace("-", ".")
if not s:
return None
m = re.fullmatch(r"(SH|SZ|BJ)\.?(\d{6})", s)
if m:
return "%s.%s" % (m.group(2), m.group(1))
m = re.fullmatch(r"(\d{6})(?:\.(SH|SZ|BJ))?", s)
if not m:
return None
code, mk = m.group(1), m.group(2)
if not mk:
if code[0] == "6":
mk = "SH"
elif code[0] == "9":
# 仅覆盖沪深京 A 股；920xxx=北交所新代码(BJ)，其余 9 头（含 900xxx 沪B）按沪兜底、不专门支持
mk = "BJ" if code.startswith("92") else "SH"
elif code[0] in "023":
mk = "SZ"
elif code[0] in "48":
mk = "BJ"
else:
mk = "SH"
return "%s.%s" % (code, mk)
# ---------------------------------------------------------------- 北交所代码转换
def resolve_query(code: str):
"""北交所老代码(83/43/8x)转为新代码 920xxx，以取到完整 F10 数据。
实测（get_misc_data 返回字段数）：
830799.BJ -> 11 个（仅基础字段） 920799.BJ -> 15 个（完整）
430047.BJ -> 11 个 920047.BJ -> 13 个
835174.BJ -> 11 个 920174.BJ -> 14 个
转换规则：920 + 原代码后 3 位（830799 -> 920799 已与用户核对）。
"""
c, mk = code.split(".")
if mk == "BJ" and len(c) == 6 and c[0] in "48":
return "920" + c[-3:] + ".BJ"
return code
# ---------------------------------------------------------------- 取数
def fetch(code: str):
from tdxaidata import tqs # 延迟导入，避免启动即失败
t0 = time.time()
qcode = resolve_query(code)
misc = None
used = qcode
def _call(c):
try:
return tqs.get_misc_data(stock_code=c)
except Exception:
return None
misc = _call(qcode)
# 新代码取不到（异常/空）时，回退老代码，至少拿到基础字段
if not misc and qcode != code:
misc = _call(code)
if misc:
used = code
if not misc:
return {"ok": False, "code": code,
"error": "该证券无综合信息资料（接口返回空），或代码无市场后缀"}
name = ""
try:
info = tqs.get_stock_info(used, ["Name"])
if isinstance(info, dict):
name = info.get("Name", "") or ""
except Exception:
pass
items = [{"k": str(k), "v": str(v)} for k, v in misc.items()]
# 头部补一行「代码名称」，与通达信 F10 面板一致
plain = code.split(".")[0]
items.insert(0, {"k": "代码名称", "v": ("%s %s" % (plain, name)).strip()})
return {"ok": True, "code": code, "query": (used if used != code else None),
"name": name, "items": items, "cost": round(time.time() - t0, 3)}
def get_data(raw_code: str):
code = normalize(raw_code)
if not code:
return {"ok": False, "error": "代码格式不正确，示例：688318 / 000001.SZ / SH600000"}
with _lock:
hit = _cache.get(code)
if hit and time.time() - hit[0] < CACHE_TTL:
return hit[1]
try:
data = fetch(code)
except ImportError:
return {"ok": False, "code": code,
"error": "未安装 tdxaidata，请执行：pip install -U tdxaidata -i https://pypi.tuna.tsinghua.edu.cn/simple"}
except Exception as exc:
return {"ok": False, "code": code, "error": "调用失败：%s" % exc}
with _lock:
_cache[code] = (time.time(), data)
return data
# ---------------------------------------------------------------- HTTP
class Handler(BaseHTTPRequestHandler):
server_version = "F10Misc/1.0"
def log_message(self, fmt, *args):
sys.stdout.write(" %s %s\n" % (time.strftime("%H:%M:%S"), fmt % args))
def _send(self, body: bytes, ctype: str, status: int = 200):
self.send_response(status)
self.send_header("Content-Type", ctype)
self.send_header("Content-Length", str(len(body)))
self.send_header("Cache-Control", "no-store")
self.send_header("Access-Control-Allow-Origin", "*")
self.end_headers()
self.wfile.write(body)
def do_GET(self):
u = urlparse(self.path)
path = u.path
if path in ("/", "/index.html"):
fp = os.path.join(BASE, "index.html")
if not os.path.exists(fp):
self._send(b"index.html missing", "text/plain; charset=utf-8", 404)
return
with open(fp, "rb") as f:
self._send(f.read(), "text/html; charset=utf-8")
return
if path == "/api/misc":
qs = parse_qs(u.query)
code = (qs.get("code") or [""])[0]
data = get_data(code)
body = json.dumps(data, ensure_ascii=False).encode("utf-8")
self._send(body, "application/json; charset=utf-8")
return
if path == "/favicon.ico":
self._send(b"", "image/x-icon", 204)
return
self._send(b'{"ok":false,"error":"not found"}', "application/json; charset=utf-8", 404)
def main():
url = "http://127.0.0.1:%d" % PORT
print("=" * 52)
print(" 通达信 F10 综合信息截图 -> %s" % url)
print(" 接口：tqs.get_misc_data | Ctrl+C 停止")
print("=" * 52)
try:
srv = ThreadingHTTPServer(("127.0.0.1", PORT), Handler)
except OSError:
print("端口 %d 被占用，换一个：set F10_PORT=8792 && python server.py" % PORT)
return
try:
srv.serve_forever()
except KeyboardInterrupt:
print("\n已停止")
finally:
srv.server_close()
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

📄

index.html

查询展示数据

```
<!DOCTYPE html>
<html lang="zh-CN" data-page-node-id="rMA5oYaIzxqGhMS9Wvmkea">
<head data-page-node-id="NzWVf0mXKH3J16LrZ20gBZ">
<meta charset="utf-8" data-page-node-id="BPo6FMyQwp3Np3JF4hnn1l">
<meta name="viewport" content="width=device-width,initial-scale=1" data-page-node-id="SjttqAD908uz0dSCh9jqII">
<title>通达信 F10 综合信息 · 截图</title>
<style>
:root{
--bg:#f4f5f7; --card:#ffffff; --line:#e3e5e9; --ink:#1b1e23; --sub:#7b8290;
--btn:#1b1e23; --btn-ink:#fff; --accent:#c8641a;
/* 通达信面板配色 */
--f10-bg:#000000;
--f10-label:#f0a63c;
--f10-value:#ffe066;
--f10-white:#f2f2f2;
--f10-green:#5ddc6a;
--f10-red:#ff6b5e;
}
*{box-sizing:border-box}
html,body{margin:0;height:100%}
body{
background:var(--bg); color:var(--ink);
font:14px/1.5 "Microsoft YaHei","PingFang SC","Hiragino Sans GB",system-ui,sans-serif;
display:flex; flex-direction:column; align-items:center;
padding:22px 16px 40px;
}
.wrap{width:100%; max-width:720px}
/* ---------- 工具条 ---------- */
.bar{
background:var(--card); border:1px solid var(--line); border-radius:12px;
padding:14px 16px; display:flex; gap:10px; align-items:center; flex-wrap:wrap;
box-shadow:0 1px 2px rgba(16,24,40,.04);
}
.title{font-weight:600; letter-spacing:.2px; margin-right:4px; white-space:nowrap}
.title small{color:var(--sub); font-weight:400; margin-left:6px; font-size:12px}
.grow{flex:1 1 auto}
input[type=text]{
width:100%; min-width:150px; height:36px; padding:0 12px;
border:1px solid var(--line); border-radius:8px; background:#fbfbfc;
font:inherit; font-variant-numeric:tabular-nums; color:var(--ink); outline:none;
transition:border-color .15s, box-shadow .15s;
}
input[type=text]:focus{border-color:#b9c0cc; box-shadow:0 0 0 3px rgba(27,30,35,.06); background:#fff}
button{
height:36px; padding:0 15px; border-radius:8px; border:1px solid var(--line);
background:#fff; color:var(--ink); font:inherit; cursor:pointer; white-space:nowrap;
transition:.15s; display:inline-flex; align-items:center; gap:6px;
}
button:hover{background:#f7f8fa; border-color:#cfd4dd}
button:active{transform:translateY(1px)}
button.primary{background:var(--btn); color:var(--btn-ink); border-color:var(--btn)}
button.primary:hover{background:#33383f}
button:disabled{opacity:.45; cursor:not-allowed; transform:none}
.meta{color:var(--sub); font-size:12px; margin:10px 2px 0}
kbd{font:11px/1 ui-monospace,monospace; background:#eef0f3; border:1px solid #dfe2e8;
border-bottom-width:2px; border-radius:4px; padding:2px 5px; color:#4a505c}
/* ---------- 通达信面板 ---------- */
.stage{margin-top:16px; display:flex; justify-content:center}
.panel{
--pad-x:16px; --rowh:25px;
background:var(--f10-bg);
color:var(--f10-value);
border-radius:6px;
padding:14px var(--pad-x);
width:460px; max-width:100%;
font-family:"Microsoft YaHei","PingFang SC",Consolas,monospace;
font-size:13.5px;
line-height:var(--rowh);
box-shadow:0 10px 30px rgba(0,0,0,.22);
overflow:hidden;
}
.rows{display:grid; grid-template-columns:max-content minmax(0,1fr); column-gap:0}
.rows .lbl{color:var(--f10-label); white-space:nowrap; padding-right:2px}
.rows .val{
color:var(--f10-value); white-space:nowrap; overflow:hidden; text-overflow:ellipsis;
padding-left:2px; cursor:default;
}
.rows .val.white{color:var(--f10-white)}
.rows .val.green{color:var(--f10-green)}
.rows .val.red{color:var(--f10-red)}
.rows .val.expanded{white-space:normal; word-break:break-all}
.rows .val.wrap{white-space:normal; word-break:break-word; overflow:visible; text-overflow:clip}
.rows .val.flash{animation:flash .55s ease}
@keyframes flash{from{background:#2a2a00}to{background:transparent}}
.panel .msg{color:#ff8a7a; line-height:1.9; white-space:pre-wrap; font-size:13px}
.panel .msg .k{color:var(--f10-label)}
.panel .loading{color:#8a8a8a}
/* ---------- 底部 ---------- */
.foot{margin-top:14px; color:var(--sub); font-size:12px; display:flex; justify-content:space-between; gap:12px; flex-wrap:wrap}
.toast{
position:fixed; left:50%; bottom:26px; transform:translate(-50%,14px);
background:#1b1e23; color:#fff; padding:9px 16px; border-radius:8px; font-size:13px;
opacity:0; pointer-events:none; transition:.22s; box-shadow:0 8px 24px rgba(0,0,0,.2); z-index:9;
}
.toast.on{opacity:1; transform:translate(-50%,0)}
</style>
</head>
<body data-page-node-id="Zeyy5xaEZSEODvTvUeZbdc">
<div class="wrap" data-page-node-id="Vi0XsB4dxeig7CXs3Z5r9K">
<div class="bar" data-page-node-id="fgayNZbWiP8BNqG5qDFehj">
<div class="title" data-page-node-id="UKh5gG7Pd6NXERz2eFtqdE">F10 综合信息<small data-page-node-id="WBzzetyC8vJ8CdBeHosnxA">get_misc_data</small></div>
<div class="grow" style="min-width:140px" data-page-node-id="NRZqYWzFO2UW4J3hlypfx8">
<input id="code" type="text" value="688318" placeholder="688318 / 000001.SZ / SH600000" spellcheck="false" data-page-node-id="wLyiC2FeFu5YbRjcK0FAzh">
</div>
<button class="primary" id="go" data-page-node-id="BFYMaWvNMM6VUmvupsWugP">查询</button>
<button id="shot" disabled data-page-node-id="iNBL8LNWDZZPFwn6Z913LK">截图 PNG</button>
<button id="copy" disabled data-page-node-id="uM4gUiRUKjGOEjJxcCaYMk">复制图片</button>
</div>
<div class="stage" data-page-node-id="xm1XcW7H0JQVI6jYM5fCrj">
<div class="panel" id="panel" data-page-node-id="trmmXLVDI160X9li6MSDO9">
<div class="msg loading" data-page-node-id="n36Wl974nbDpc4JNrPftgU">等待查询…</div>
</div>
</div>
<div class="meta" data-page-node-id="OO7jmqgUNSH8KLQ4YjG3Se">
回车即可查询 · 数值为接口原样返回的格式化文本 · 长内容自动换行 · 短内容点击可展开
</div>
<div class="foot" data-page-node-id="mH6fpxTygEDavlQFhQMsgf">
<span data-page-node-id="lIpXaC2KQkRXM9XCzMstBE">接口：<b data-page-node-id="EwU5aGJM8rAExVFmQfdvlM">tqs.get_misc_data</b> · 范围：沪深京品种 · 数据源 TdxAiData</span>
<span data-page-node-id="cClcAkQxDtCkJnKkQlrAWy"><b id="count" data-page-node-id="RFoPrRjcUPs4S6jkStx9lN"></b> <span id="cost" data-page-node-id="jSEBlb51mGTmMiJtCpUdPX"></span></span>
</div>
</div>
<div class="toast" id="toast" data-page-node-id="E839QrwA81JQNEN2IFuD91"></div>
<script>
const API = 'http://127.0.0.1:8791';
const FONT = '13.5px "Microsoft YaHei","PingFang SC",Consolas,monospace';
const LBL_COLOR = '#f0a63c', VAL_COLOR = '#ffe066', WHITE = '#f2f2f2', GREEN = '#5ddc6a', RED = '#ff6b5e';
const $ = s => document.querySelector(s);
const panel = $('#panel'), input = $('#code');
let current = null; // {code,name,items}
// ---------- 工具 ----------
function toast(msg){
const t = $('#toast');
t.textContent = msg; t.classList.add('on');
clearTimeout(t._t); t._t = setTimeout(()=>t.classList.remove('on'), 1800);
}
// 风险类栏目：A 股红涨绿跌 ——「无 / 0 / 未来三个月无」= 利好(红)，有质押/减持/解禁 = 利空(绿)
const RISK_KEYS = ['总质押数','占总股本比','减持计划','最近解禁'];
function tone(k, v){
if (k === '代码名称') return 'white';
if (RISK_KEYS.includes(k)) return /^(0|无|未来)/.test(v) ? 'red' : 'green';
return '';
}
// 同类型、各自很短的字段合并到一行（如 总质押数 + 占总股本比）
const MERGE = [
{keys:['总质押数','占总股本比'], maxLen:30,
fmt:(a,b)=> a + '（占总股本 ' + b + '）'},
{keys:['减持计划','最近解禁'], maxLen:38,
fmt:(a,b)=> a + ' · 最近解禁 ' + b},
];
// 文字较多的栏目：默认换行（不截断）；其余短栏目超长才换行
const LONG_KEYS = new Set(['最新涨停','异动原因','最新跌停','主营业务','最新公告',
'互动问答','龙虎榜单','合计减持','最新计划','新增概念','所属板块','所属主题',
'行业地位','行业地域','地区','北向成交','做市信息','融资买入']);
function buildRenderItems(items){
const map = new Map(items.map(it=>[it.k, it]));
const consumed = new Set();
const merges = [];
for (const rule of MERGE){
const a = map.get(rule.keys[0]), b = map.get(rule.keys[1]);
if (a && b){
const v = rule.fmt(a.v, b.v);
if (v.length <= rule.maxLen){ // 都短才合并，否则各自成行
const ia = items.findIndex(it => it.k === rule.keys[0]);
const ib = items.findIndex(it => it.k === rule.keys[1]);
merges.push({at: Math.min(ia, ib), k: a.k, v});
consumed.add(rule.keys[0]); consumed.add(rule.keys[1]);
}
}
}
const out = [];
for (let i = 0; i < items.length; i++){
const m = merges.find(x => x.at === i);
if (m) out.push({k:m.k, v:m.v}); // 合并行放在该对字段最早出现的位置
if (consumed.has(items[i].k)) continue; // 原始字段不再单独输出
out.push({...items[i]});
}
return out;
}
function shouldWrap(it){ return LONG_KEYS.has(it.k) || it.v.length > 36; }
// ---------- 渲染 ----------
function render(data){
if (!data.ok){
panel.innerHTML = '';
const d = document.createElement('div');
d.className = 'msg';
d.innerHTML = '<span class="k">' + (data.code || '') + '</span>\n' + (data.error || '查询失败');
panel.appendChild(d);
$('#shot').disabled = $('#copy').disabled = true;
$('#count').textContent = '';
$('#cost').textContent = '';
return;
}
current = {code:data.code, name:data.name, query:data.query, items: buildRenderItems(data.items)};
panel.innerHTML = '';
const rows = document.createElement('div');
rows.className = 'rows';
for (const it of current.items){
const l = document.createElement('div');
l.className = 'lbl'; l.textContent = it.k + '：';
const v = document.createElement('div');
const w = shouldWrap(it);
v.className = 'val ' + tone(it.k, it.v) + (w ? ' wrap' : '');
v.textContent = it.v;
if (!w){ // 短栏目：超长截断，点击展开
v.title = it.k + '：' + it.v;
v.addEventListener('click', ()=> v.classList.toggle('expanded'));
}
rows.append(l, v);
}
panel.appendChild(rows);
$('#shot').disabled = $('#copy').disabled = false;
$('#count').textContent = '共 ' + current.items.length + ' 个栏目';
$('#cost').textContent = (data.cost ? '· 耗时 ' + data.cost + 's' : '');
// 北交所老代码转新代码查询时，提示一句（不进截图）
let meta = '回车即可查询 · 数值为接口原样返回的格式化文本 · 长内容自动换行 · 短内容点击可展开';
if (data.query && data.query !== data.code){
meta += ' · 北交所已用新代码 ' + data.query.split('.')[0] + ' 查询完整数据';
}
$('.meta').textContent = meta;
}
// ---------- 查询 ----------
async function query(){
const code = input.value.trim();
if (!code){ input.focus(); return; }
panel.innerHTML = '<div class="msg loading">查询中…</div>';
$('#shot').disabled = $('#copy').disabled = true;
const t0 = performance.now();
try{
const r = await fetch(API + '/api/misc?code=' + encodeURIComponent(code));
const j = await r.json();
if (!j.cost) j.cost = ((performance.now() - t0)/1000).toFixed(2);
render(j);
}catch(e){
render({ok:false, code:code, error:'无法连接本地服务 ' + API + '\n请先运行 启动服务.bat（或 python server.py）'});
}
}
$('#go').onclick = query;
input.addEventListener('keydown', e => { if (e.key === 'Enter') query(); });
// ---------- Canvas 截图 ----------
// 按字符宽度换行（适配中文长文本）
function wrapLines(ctx, text, maxW){
const lines = []; let line = '';
for (const ch of text){
if (line && ctx.measureText(line + ch).width > maxW){ lines.push(line); line = ch; }
else line += ch;
}
if (line) lines.push(line);
return lines.length ? lines : [''];
}
function buildCanvas(){
const padX = 16, padY = 14, rowh = 25;
const c = document.createElement('canvas');
const ctx = c.getContext('2d');
ctx.font = FONT;
// 标签列宽
let lblW = 0;
for (const it of current.items) lblW = Math.max(lblW, ctx.measureText(it.k + '：').width);
// 面板宽度 = 实际显示宽度（保证截图与所见一致）
const W = Math.ceil(panel.getBoundingClientRect().width);
const innerW = W - padX * 2 - lblW - 10;
// 先算每行的高度与文字（换行 or 截断）
const layout = current.items.map(it => {
if (shouldWrap(it)){
const lines = wrapLines(ctx, it.v, innerW);
return {it, lines, h: Math.max(1, lines.length) * rowh};
}
let s = it.v;
if (ctx.measureText(s).width > innerW){
while (s.length > 1 && ctx.measureText(s + '…').width > innerW) s = s.slice(0, -1);
s += '…';
}
return {it, lines:[s], h: rowh};
});
const H = padY * 2 + layout.reduce((a, b) => a + b.h, 0);
c.width = W; c.height = H;
ctx.font = FONT;
ctx.textBaseline = 'middle';
// 背景
ctx.fillStyle = '#000'; ctx.fillRect(0, 0, W, H);
// 逐行绘制
let y = padY;
for (const L of layout){
const it = L.it;
const firstY = y + rowh / 2;
ctx.textAlign = 'left';
ctx.fillStyle = LBL_COLOR;
ctx.fillText(it.k + '：', padX, firstY);
const t = tone(it.k, it.v);
ctx.fillStyle = t === 'white' ? WHITE : (t === 'red' ? RED : (t === 'green' ? GREEN : VAL_COLOR));
const x = padX + lblW + 4;
L.lines.forEach((ln, idx) => ctx.fillText(ln, x, y + idx * rowh + rowh / 2));
y += L.h;
}
return c;
}
function fileName(){
const d = new Date();
const p = n => String(n).padStart(2, '0');
const stamp = `${d.getFullYear()}${p(d.getMonth()+1)}${p(d.getDate())}_${p(d.getHours())}${p(d.getMinutes())}`;
return `${current.code.replace('.', '')}_${current.name || 'F10'}_${stamp}.png`;
}
$('#shot').onclick = async () => {
if (!current || !current.ok) return;
await document.fonts.ready;
const c = buildCanvas();
const scale = 2; // 2x 高清
const out = document.createElement('canvas');
out.width = c.width * scale; out.height = c.height * scale;
const octx = out.getContext('2d');
octx.imageSmoothingEnabled = true;
octx.drawImage(c, 0, 0, out.width, out.height);
out.toBlob(blob => {
const a = document.createElement('a');
a.href = URL.createObjectURL(blob);
a.download = fileName();
a.click();
setTimeout(()=>URL.revokeObjectURL(a.href), 4000);
toast('已保存 ' + out.width + '×' + out.height + ' PNG');
}, 'image/png');
};
$('#copy').onclick = async () => {
if (!current || !current.ok) return;
await document.fonts.ready;
const c = buildCanvas();
try{
const blob = await new Promise(r => c.toBlob(r, 'image/png'));
await navigator.clipboard.write([new ClipboardItem({'image/png': blob})]);
toast('图片已复制到剪贴板');
}catch(e){
toast('当前浏览器不允许写剪贴板，请用「截图 PNG」');
}
};
// 启动即查一次
window.addEventListener('load', query);
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
