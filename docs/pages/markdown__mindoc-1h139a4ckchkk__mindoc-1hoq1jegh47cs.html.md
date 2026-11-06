---
title: "删除自定义板块成份股deluserblock"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1h139a4ckchkk/mindoc-1hoq1jegh47cs.html
fetched_at: 2026-11-06T06:05:14Z
---

### # 删除自定义板块成份股del_user_block

#### # 从指定的自定义板块中移除一只或多只证券，只删证券不删板块

```
del_user_block(
block_code: str,
stock_list: List[str],
show: bool = False
) -> Dict
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
| block_code | Y | str | 自建板块简称，如 CSBK |
| stock_list | Y | List[str] | 要移除的证券代码列表，如 ["688318.SH"]；传空列表会直接返回空字典 |
| show | N | bool | 是否在客户端显示该板块，默认 False |

#### # 返回数据

| 数据字段 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| ErrorId | Y | str | 错误码，0 表示成功 |
| Msg | Y | str | 执行结果说明，成功时为 Del User Block Completed |
| run_id | Y | str | 本次策略运行编号 |

成功返回含 ErrorId、Msg、run_id 三个字段的字典，不含 Value；stock_list 为空时返回空字典。


#### # 接口使用

从自建板块 CSBK 中移除 688318.SH，板块本身保留。

```
from tqcenter import tq
tq.initialize(__file__)
res = tq.del_user_block(
block_code="CSBK",
stock_list=["688318.SH"],
show=False,
)
print(res)
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
{"ErrorId": "0", "Msg": "Del User Block Completed", "run_id": "5"}
```


1
