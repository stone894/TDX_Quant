---
title: "批量获取集合竞价getcallauction_batch"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hjbidnohpjn4/mindoc-1hlsf010md5gg.html
fetched_at: 2026-11-06T06:03:20Z
---

### # 批量获取集合竞价get_call_auction_batch

按证券代码列表批量获取集合竞价时序，逐只返回虚拟匹配价、虚拟匹配量、未匹配量；type=0 返回早盘与尾盘两段，type=1 只返回早盘

范围：2025年以来的所有沪深京品种

```
get_call_auction_batch(
stock_list: List[str] = [],
field_list: List[str] = [],
type: int = 1,
date: str = "",
return_df: bool = False
) -> dict | DataFrame
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

#### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| stock_list | Y | List[str] | 证券代码列表，须带市场后缀，如 ["600000.SH", "000001.SZ"] |
| field_list | N | List[str] | 需要返回的字段名列表，传空返回全部字段 |
| type | N | int | 竞价范围：0 早盘和尾盘、1 仅早盘（默认） |
| date | N | str | 查询日期，支持 YYYYMMDD、YYYY-MM-DD；传空取当日 |
| return_df | N | bool | 为 True 且已安装 pandas 时返回 DataFrame |

#### # 返回数据

| 数据字段 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| 证券代码 | Y | dict | 键为入参中的证券代码，值为该证券的竞价字段字典 |
| Time | Y | List[str] | 集合竞价时刻，格式 HHMMSS，如 091502 |
| Price | Y | List[str] | 该时刻的虚拟匹配价 |
| Volume | Y | List[str] | 该时刻的虚拟匹配量 |
| LeaveQty | Y | List[str] | 该时刻的未匹配量，可为负值 |


`field_list`

非空时，每只证券的字典中只保留指定字段。

已安装 pandas 时`return_df=True`

返回 DataFrame，行索引为证券代码、列名为竞价字段名，每个单元格为该证券对应的取值数组；未安装 pandas 时始终返回字典。

#### # 接口使用

取 600000.SH、000001.SZ 两只证券在 2026-10-16 的早盘集合竞价时序。

```
from tdxaidata import tqs
data = get_call_auction_batch(
stock_list=["600000.SH", "000001.SZ"],
field_list=[],
type=1,
date="2026-10-16",
)
print(data)
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
{
"600000.SH": {
"LeaveQty": ["181", "176", "236", "202", "178"],
"Price": ["9.60", "9.60", "9.60", "9.60", "9.60"],
"Time": ["091502", "091505", "091511", "091514", "091526"],
"Volume": ["272.00", "272.00", "272.00", "272.00", "272.00"]
},
"000001.SZ": {
"LeaveQty": ["914", "-103", "248", "116", "-127"],
"Price": ["11.30", "11.40", "11.40", "11.41", "11.43"],
"Time": ["091500", "091509", "091518", "091527", "091536"],
"Volume": ["1549.00", "2355.00", "2458.00", "2577.00", "2739.00"]
}
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
