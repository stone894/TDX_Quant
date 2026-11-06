---
title: "获取自定义板块列表get_user_sector"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1h139a4ckchkk/mindoc-1h1hauh9inaac.html
fetched_at: 2026-11-06T06:05:12Z
---

# 获取自定义板块列表get_user_sector

### 获取自定义板块代码列表

```
get_user_sector(cls) -> List:
```


1


### 接口使用

```
from tqcenter import tq
tq.initialize(__file__)
user_list = tq.get_user_sector()
print(user_list)
print(len(user_list))
```


1

2

3

4

5


### 数据样本

```
[{'Code': 'CSBK', 'Name': '测试板块'}, {'Code': 'CSBK2', 'Name': '测试板块2'}]
```


1
