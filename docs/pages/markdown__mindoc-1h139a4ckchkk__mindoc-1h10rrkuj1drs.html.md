---
title: "创建自定义板块"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1h139a4ckchkk/mindoc-1h10rrkuj1drs.html
fetched_at: 2026-11-06T06:05:17Z
---

# # 创建自定义板块

### # 在通达信客户端中创建自定义板块

```
create_sector(block_code:str = '',
block_name:str = ''):
```


1

2


2

### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| block_code | Y | str | 自定义板块简称 |
| block_name | Y | str | 自定义板块名称 |

注：block_code一般为板块名称中的每个汉字的声母组成（如果是自定义板块名称中有汉字的话）

### # 接口使用

```
from tqcenter import tq
tq.initialize(__file__)
create_ptr = tq.create_sector(block_code='CSBK2', block_name='测试板块2')
print(create_ptr)
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
{
"Error" : "创建CSBK2板块成功",
"ErrorId" : "0",
"run_id" : "1"
}
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
