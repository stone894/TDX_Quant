---
title: "批量计算多标的基础选股因子"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hho7blr2j340/mindoc-1hhoaleti08f0/mindoc-1hhoantm89gu4.html
fetched_at: 2026-11-06T06:05:21Z
---

### # 批量计算多标的基础选股因子

```
calc_factor_batch(
symbols,
factors=None,
count=20,
frequency="1d",
end_date=None,
fq=None,
)
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

批量计算多标的基础选股因子，返回以证券代码为索引的 `DataFrame`

。

当前支持因子：

| 因子 | 别名 | 说明 |
|---|---|---|
`return` | `returns` 、`ret` 、`momentum` | 区间首尾收盘价收益率 |
`ma` | `mean_close` 、`close_mean` | 区间收盘价均值 |
`volatility` | `vol` 、`std` | 区间收益率标准差 |
`volume_mean` | `avg_volume` | 区间成交量均值 |
`close` | `last_close` | 最新收盘价 |

选股示例：

```
factors = calc_factor_batch(
["000001.SH", "600000.SH"],
["return", "ma", "volatility", "volume_mean", "close"],
count=20,
)
selected = list(factors.sort_values("return", ascending=False).head(2).index)
weights = equal_weight(selected, total_percent=0.6)
order_target_weights(weights, close_missing=True)
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


2

3

4

5

6

7

8

9
