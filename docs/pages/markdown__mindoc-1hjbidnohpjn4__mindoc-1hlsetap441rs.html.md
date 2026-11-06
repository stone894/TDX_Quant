---
title: "获取分笔行情gettickdata"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hjbidnohpjn4/mindoc-1hlsetap441rs.html
fetched_at: 2026-11-06T06:03:13Z
---

### # 获取分笔行情get_tick_data

获取单只证券指定日期的分笔成交明细，按笔返回成交时间、价格、成交量和买卖方向

范围：2000年以来的所有沪深京品种

```
get_tick_data(
stock_code: str,
date: str,
startxh: int = 0,
wantnum: int = 0
) -> DataFrame | dict
```


1

2

3

4

5

6


2

3

4

5

6

#### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| stock_code | Y | str | 单只证券代码，需带市场后缀，如 `000001.SZ` |
| date | Y | str | 查询日期，支持 `YYYYMMDD` 、`YYYY-MM-DD` 、`YYYY-MM-DD HH:MM:SS` 三种格式，如 `2026-08-21` |
| startxh | N | int | 起始分笔序号，从 `0` 开始，用于翻页，默认 0 |
| wantnum | N | int | 本次请求条数，默认 0；翻页时按固定页长取值，如 `100` |

#### # 返回数据

已安装 pandas 时返回 DataFrame，列为下表的四个字段，行索引为默认整数索引，一行表示一笔成交，各列均为字符串。

未安装 pandas 时返回原始字典，四个字段为长度一致的字符串数组，并多一个 `TotalNum`

表示本次返回的记录数。

| 数据字段 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| Time | Y | str | 成交时间，格式为 `HHMMSS` ，如 `093000` |
| Price | Y | str | 成交价格 |
| Volume | Y | str | 成交量 |
| BSFlag | Y | str | 买卖方向标识，实际枚举含义以数据服务定义为准 |
| TotalNum | N | int | 本次实际返回的记录数，仅未安装 pandas 时随字典返回 |

#### # 接口使用

获取 `000001.SZ`

在 2026-08-21 的首批分笔成交数据。

```
from tdxaidata import tqs
ticks = get_tick_data(
stock_code="000001.SZ",
date="2026-08-21",
startxh=0,
wantnum=100,
)
print(ticks)
```


1

2

3

4

5

6

7

8


2

3

4

5

6

7

8

#### # 数据样本

```
Time Price Volume BSFlag
0 092500 11.36 3049 2
1 093000 11.36 1737 1
2 093003 11.37 1547 0
```


1

2

3

4


2

3

4
