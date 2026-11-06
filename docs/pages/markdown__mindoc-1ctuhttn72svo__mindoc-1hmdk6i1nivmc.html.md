---
title: "获取扩展市场板块代码列表get_sector_list_ds"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1ctuhttn72svo/mindoc-1hmdk6i1nivmc.html
fetched_at: 2026-11-06T06:05:11Z
---

# # 获取扩展市场板块代码列表get_sector_list_ds

### # 获取扩展市场全部板块代码列表

```
def get_sector_list_ds(list_type: int = 0) -> List:
```


1


### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| list_type | Y | int | 返回数据类型 |

- list_type = 0 只返回代码，list_type = 1 返回代码和名称

### # 接口使用

```
from tqcenter import tq
tq.initialize(__file__)
block_list = tq.get_sector_list_ds()
print(block_list)
block_list2 = tq.get_sector_list_ds(list_type = 1)
print(block_list2)
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

注：此接口相当于 get_stock_list('110')

### # 数据样本

```
['HK0201.UZ', 'HK0202.UZ', 'HK0203.UZ', 'HK0204.UZ', 'HK0205.UZ', 'HK0206.UZ', 'HK0207.UZ', 'HK0208.UZ', 'HK0209.UZ', 'HK0210.UZ', 'HK0211.UZ', ...]
[{'Code': 'HK0201.UZ', 'Name': '港股-内地房地产'}, {'Code': 'HK0202.UZ', 'Name': '港股-台资'}, {'Code': 'HK0203.UZ', 'Name': '港股-太阳能'}, {'Code': 'HK0204.UZ', 'Name': '港股-风电'}, {'Code': 'HK0205.UZ', 'Name': '港股-环保'}, {'Code': 'HK0206.UZ', 'Name': '港股-高铁'}, ...]
```


1

2

3


2

3
