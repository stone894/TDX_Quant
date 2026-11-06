---
title: "获取日线资金统计数据get_exday_data"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1ctuhthaq5qmg/mindoc-1hh9gktmqsv5c.html
fetched_at: 2026-11-06T06:04:11Z
---

# # 获取日线资金统计数据get_exday_data

### # 获取指定股票的日线资金统计数据

```
def get_exday_data(stock_code: str = '',
count: int = 1):
```


1

2


2

### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| stock_code | Y | str | 股票代码 |
| count | N | int | 返回最近记录数量，默认 `1` |

- 此函数需要通达信客户端有专业分析权限（专业版等）

### # 输出数据

| 名称 | 类型 | 说明 |
|---|---|---|
| Date | str | 日期 |
| CJBS | str | 成交单数 |
| Vol | List[List[str]] | 4×4 成交数量（手），按特大/大/中/小和买入/卖出/主买/主卖 |
| Amo | List[List[str]] | 4×4 成交金额（元），按特大/大/中/小和买入/卖出/主买/主卖 |
| VolNum | List[List[str]] | 2×2 成交笔数（笔），(特大+大)/(中+小),买/卖二类 |
| BOrder / BCancel | str | 累计买净挂单 / 买净撤单 |
| SOrder / SCancel | str | 累计卖净挂单 / 卖净撤单 |
| BuyAvp / SellAvp | str | 委买均价 / 委卖均价 |
| TotalBOrder / TotalSOrder | str | 当前总委买 / 当前总委卖 |

### # 接口使用

```
from tqcenter import tq
tq.initialize(__file__)
exday_data = tq.get_exday_data(stock_code='688318.SH', count = 1)
print(exday_data)
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
[{'Date': '20260724', 'CJBS': '11214.00', 'Vol': [['3248.06', '6216.93', '1370.62', '2506.81'], ['8844.36', '11588.53', '2804.48', '6758.76'], ['18149.82', '18483.47', '7522.38', '11017.52'], ['18393.98', '12347.28', '9196.50', '7457.20']], 'Amo': [['21479328.00', '41183188.00', '9076769.00', '16590353.00'], ['58281736.00', '76518904.00', '18507346.00', '44656932.00'], ['120153008.00', '122070008.00', '49816088.00', '72774976.00'], ['121716192.00', '81858176.00', '60835584.00', '49365788.00']], 'VolNum': [['170.90', '245.59'], ['7048.90', '4888.86']], 'BOrder': '126245.16', 'BCancel': '76479.00', 'SOrder': '136949.16', 'SCancel': '82938.00', 'BuyAvp': '0.00', 'SellAvp': '0.00', 'TotalBOrder': '1128.00', 'TotalSOrder': '5373.00'}]
```


1
