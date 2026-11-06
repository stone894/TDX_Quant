---
title: "获取股票标记getstockmark"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1h139a4ckchkk/mindoc-1hnnu606ehjd8.html
fetched_at: 2026-11-06T06:05:07Z
---

### # 获取股票标记get_stock_mark

#### # 获取指定证券的标记信息，逐只返回标记标志位与标记文字

```
get_stock_mark(
stock_list: List[str]
) -> DataFrame
```


1

2

3


2

3

#### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| stock_list | Y | List[str] | 证券代码列表，须带市场后缀，如 ["688318.SH", "600006.SH"] |

#### # 返回数据

| 数据字段 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| mark_flag | Y | int | 标记标志位，未设置标记时为 0 |
| mark_text | Y | str | 标记文字，未设置标记时为空字符串 |

返回 DataFrame，行索引为证券代码，列固定为 mark_flag、mark_text；行序以返回结果为准。


未设置标记的证券同样占一行，其 mark_flag 为 0、mark_text 为空串。

stock_list 为空、或代码格式非法时，返回同列名的空 DataFrame（0 行）。

#### # 接口使用

取 688318.SH、600006.SH 两只证券的标记信息。

```
from tqcenter import tq
mk = tq.get_stock_mark(
stock_list=['688318.SH', '600006.SH'],
)
print(mk)
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

#### # 数据样本

```
mark_flag mark_text
600006.SH 7 东风股份测试标记
688318.SH 7 好股票人人爱
```


1

2

3


2

3
