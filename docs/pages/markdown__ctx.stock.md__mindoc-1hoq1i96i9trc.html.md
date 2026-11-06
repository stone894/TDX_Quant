---
title: "获取版本信息get_version"
source_url: https://help.tdx.com.cn/quant/docs/markdown/ctx.stock.md/mindoc-1hoq1i96i9trc.html
fetched_at: 2026-11-06T06:03:05Z
---

### # 获取版本信息get_version

#### # 返回当前 tqcenter 模块的版本号与版本日期，可在 initialize 之前调用

```
get_version() -> Dict[str, str]
```


1


#### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| 无 | — | — | 该接口无入参 |

#### # 返回数据

| 数据字段 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| name | Y | str | 模块名，固定为 tqcenter |
| version | Y | str | 版本号，如 1.3.1 |
| date | Y | str | 版本日期，如 2026-10-23 |

#### # 接口使用

查询当前加载的 tqcenter 模块版本。

```
from tqcenter import tq
info = tq.get_version()
print(info)
```


1

2

3


2

3

#### # 数据样本

```
{"name": "tqcenter", "version": "1.3.1", "date": "2026-10-23"}
```


1
