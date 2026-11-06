---
title: "按数量下单"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hho7blr2j340/mindoc-1hhobbn0nuedg/mindoc-1hhobbvteo4ds.html
fetched_at: 2026-11-06T06:05:01Z
---

### # 按数量下单

```
order(symbol, amount, price=None)
```


1


参数：

| 参数 | 说明 |
|---|---|
`symbol` | 证券代码 |
`amount` | 数量，正数买入，负数卖出 |
`price` | 限价价格；`None` 表示市价 |

示例：

```
order("000001.SZ", 1000)
order("000001.SZ", -500)
```


1

2


2

← 时间函数 按金额下单order_value →

```
order(symbol, amount, price=None)
```


参数：

| 参数 | 说明 |
|---|---|
`symbol` | 证券代码 |
`amount` | 数量，正数买入，负数卖出 |
`price` | 限价价格；`None` 表示市价 |

示例：

```
order("000001.SZ", 1000)
order("000001.SZ", -500)
```


← 时间函数 按金额下单order_value →
