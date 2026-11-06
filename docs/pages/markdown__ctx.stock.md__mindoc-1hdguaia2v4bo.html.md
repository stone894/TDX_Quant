---
title: "检索证券信息get_match_stkinfo"
source_url: https://help.tdx.com.cn/quant/docs/markdown/ctx.stock.md/mindoc-1hdguaia2v4bo.html
fetched_at: 2026-11-06T06:03:04Z
---

# # 检索证券信息get_match_stkinfo

### # 检索证券信息

```
def get_match_stkinfo(key_word:str = ''):
```


1


### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| key_word | Y | str | 关键词 |

### # 输出数据

| 名称 | 类型 | 数值 | 说明 |
|---|---|---|---|
| Code | Y | str | 证券代码 |
| Name | Y | str | 证券名称 |

### # 接口使用

```
from tqcenter import tq
tq.initialize(__file__)
match_stkinfo = tq.get_match_stkinfo(key_word='通达信')
print(match_stkinfo)
```


1

2

3

4


2

3

4

### # 数据样本

```
[{'Code': '880515.SH', 'Name': '通达信88'},
{'Code': '880818.SH', 'Name': '通达信热股'},
{'Code': '688318.SH', 'Name': '财富趋势'}]
```


1

2

3


2

3
