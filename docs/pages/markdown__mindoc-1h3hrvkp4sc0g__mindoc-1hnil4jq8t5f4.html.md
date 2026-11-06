---
title: "批量运行通达信公式文件formulabyfile"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1h3hrvkp4sc0g/mindoc-1hnil4jq8t5f4.html
fetched_at: 2026-11-06T06:04:03Z
---

### # 批量运行通达信公式文件formula_by_file

从本地文本文件读取临时公式源码批量计算股票池内各证券的公式输出。

注：如是是tqcenter本地模式；如果有Level2行情，才能支持分钟线周期的公式计算。

注：如果是tqserver/tdxaidata模式，才支持5s,10s,15s,20s,fst,auc等周期的公式计算。

```
formula_by_file(
formula_file: str,
domain:int = 0,
data_source:int = 0,
stock_list: List[str],
stock_period: str = '1d',
start_time: str = '',
end_time: str = '',
count: int = 0,
return_count: int = 1,
return_date: bool = False,
xsflag: int = -1,
dividend_type: int = 0,
formula_type: int = 0
) -> Dict
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

#### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| formula_file | Y | str | 临时公式源码文件绝对路径（本地文本文件，仅支持txt格式），文件内容为公式源码，如 `UP3:UPNDAY(CLOSE,3);` |
| domain | Y | int | 排序域，可选值见`get_domain_sorthq` |
| data_source | N | int | 数据源，0 沪深京（默认）、1 扩展行情 |
| stock_list | Y | List[str] | 证券代码列表，需带市场后缀，如 `['000001.SZ', '600519.SH']` |
| stock_period | N | str | K 线周期，默认 `1d` ；`5m` 、`1w` 、`1mon` 等周期串同样可用，取值非法时返回空结果 |
| start_time | N | str | 起始时间，支持 `YYYYMMDD` 、`YYYY-MM-DD` 、`YYYYMMDDHHMMSS` 、`YYYY-MM-DD HH:MM:SS` 四种格式 |
| end_time | N | str | 结束时间，格式同 start_time |
| count | N | int | 行情数据条数，默认 0；大于 0 时取最近 n 条 K 线；`-1` 取全部；`-2` 使用无序列数据；为 0 时按 start_time / end_time 区间取数，两者都不传则等同 `-1` |
| return_count | N | int | 每只证券返回的结果条数，默认 1，取最近 n 条；填 `0` 返回全部 |
| return_date | N | bool | 是否同时返回每条结果的日期，默认 False |
| xsflag | N | int | 小数位控制，默认 -1 使用公式默认精度，`0` ~`8` 可指定小数位数 |
| dividend_type | N | int | 复权类型：`0` 不复权（默认）、`1` 前复权、`2` 后复权 |
| formula_type | N | int | 公式类型：`0` 技术指标公式（默认）、`1` 条件选股公式 |

- domain和data_source仅在tdxaidata有效，tq无此入参，domain为0时，stock_list最多可传入500只股票代码，否则不生效。

#### # 返回数据

| 数据字段 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| ErrorId | Y | str | 错误码，`0` 表示成功 |
| <证券代码> | Y | dict | 键为证券代码，如 `000001.SZ` ，值为该证券各输出线的结果 |
| └ <输出线名> | Y | List | 公式的输出线，键名与公式源码中定义的名称一致，支持中文，如 `UP3` |
| └ └ Date | N | str | 结果对应的日期，格式 `YYYYMMDD` ，仅 return_date=True 时返回 |
| └ └ Value | N | str | 结果的值，小数位受 xsflag 控制，仅 return_date=True 时返回 |
| Error | N | dict | 个别证券计算失败时的原因，键为证券代码 |

return_date=True 时每条结果为

`{"Date": "...", "Value": "..."}`

；return_date=False 时直接为该输出线的值数组、不再带日期，如`{"000001.SZ": {"UP3": ["1.000", "0.000"]}}`

。

formula_type=1（条件选股）只返回在取数窗口内至少有一条结果为真的证券，窗口内完全没有命中的证券不会出现在返回结果中；formula_type=0 返回股票池内全部证券。


参数不合法、公式文件读取失败或公式编译失败时返回空结构

`{}`

；个别证券出错时该证券不返回数据，原因放在`Error`

中（此时 ErrorId 仍为`0`

）。返回结构与 formula_by_src 完全一致。

#### # 接口使用

读取本地公式文件 UPN.txt，对全市场 A 股批量计算，取每只证券最近 5 个结果并带日期。

```
from tqcenter import tq
tq.initialize(__file__)
all_stocks = tq.get_stock_list(market='5')
result = tq.formula_by_file(
formula_file='F:\\tdx\\new_tdx_600\\PYPlugins\\UPN.txt',
stock_list=all_stocks,
stock_period='1d',
count=100,
return_count=1,
return_date=True,
xsflag=3,
dividend_type=0,
formula_type=1
)
print(result)
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

#### # 数据样本

```
{
"000001.SZ": {
"UP3": [
{"Date": "20261008", "Value": "1.000"},
{"Date": "20261009", "Value": "0.000"},
{"Date": "20261012", "Value": "0.000"},
{"Date": "20261013", "Value": "0.000"},
{"Date": "20261014", "Value": "0.000"}
]
},
"000333.SZ": {
"UP3": [
{"Date": "20261008", "Value": "0.000"},
{"Date": "20261009", "Value": "0.000"},
{"Date": "20261012", "Value": "1.000"},
{"Date": "20261013", "Value": "0.000"},
{"Date": "20261014", "Value": "0.000"}
]
},
"000651.SZ": {
"UP3": [
{"Date": "20261008", "Value": "0.000"},
{"Date": "20261009", "Value": "1.000"},
{"Date": "20261012", "Value": "1.000"},
{"Date": "20261013", "Value": "1.000"},
{"Date": "20261014", "Value": "1.000"}
]
},
"ErrorId": "0"
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
