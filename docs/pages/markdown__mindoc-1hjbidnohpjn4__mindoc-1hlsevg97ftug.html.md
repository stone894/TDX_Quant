---
title: "获取集合竞价getcallauction"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hjbidnohpjn4/mindoc-1hlsevg97ftug.html
fetched_at: 2026-11-06T06:03:19Z
---

### # 获取集合竞价get_call_auction

获取单只证券的集合竞价时序，含虚拟匹配价、虚拟匹配量、未匹配量；type=0 返回早盘与尾盘两段，type=1 只返回早盘

范围：2025年以来的所有沪深京品种

```
get_call_auction(
stock_code: str,
type: int = 1,
date: str = ""
) -> DataFrame | dict
```


1

2

3

4

5


2

3

4

5

#### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| stock_code | Y | str | 单只证券代码，须带市场后缀，如 600000.SH |
| type | N | int | 竞价范围：0 早盘和尾盘、1 仅早盘（默认） |
| date | N | str | 查询日期，支持 YYYYMMDD、YYYY-MM-DD；传空取当日 |

#### # 返回数据

| 数据字段 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| Time | Y | List[str] | 集合竞价时刻，格式 HHMMSS，如 091502 |
| Price | Y | List[str] | 该时刻的虚拟匹配价 |
| Volume | Y | List[str] | 该时刻的虚拟匹配量 |
| LeaveQty | Y | List[str] | 该时刻的未匹配量，可为负值 |

已安装 pandas 时返回 DataFrame，列名同上（列序 LeaveQty、Price、Time、Volume），每列均为字符串；未安装 pandas 时返回等长字符串数组组成的字典。


#### # 接口使用

取 600000.SH 在 2026-10-16 的早盘集合竞价时序。

```
from tdxaidata import tqs
auction = get_call_auction(
stock_code="600000.SH",
type=1,
date="2026-10-16",
)
print(auction)
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

#### # 数据样本

```
{
"LeaveQty": ["181", "176", "236", "202", "178"],
"Price": ["9.60", "9.60", "9.60", "9.60", "9.60"],
"Time": ["091502", "091505", "091511", "091514", "091526"],
"Volume": ["272.00", "272.00", "272.00", "272.00", "272.00"]
}
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
