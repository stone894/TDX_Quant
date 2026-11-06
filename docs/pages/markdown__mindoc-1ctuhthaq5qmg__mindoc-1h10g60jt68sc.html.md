---
title: "获取K线行情get_market_data"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1ctuhthaq5qmg/mindoc-1h10g60jt68sc.html
fetched_at: 2026-11-06T06:04:02Z
---

# # 获取K线行情get_market_data

### # 根据股票，获取历史行情

注：如是是tqcenter本地模式，支持的周期数据个数为客户端盘后数据下载的数据个数；如果有Level2行情，才能获取分钟线周期的数据。

注：如果是tqserver/tdxaidata从服务端取数据，支持全部日线以上周期的数据，支持2024年9月至今的1m分钟线，支持2017年1月至今的5m,15m,30m,1h等分钟线。如果取较长时间段的分钟周期数据，可能会消耗较多的积分。

```
get_market_data(field_list: List[str] = [],
stock_list: List[str] = [],
period: str = '',
start_time: str = '',
end_time: str = '',
count: int = -1,
dividend_type: Optional[str] = None,
fill_data: bool = True) -> Dict:
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

### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| field_list | N | List[str] | 字段筛选，传空则返回全部 |
| stock_list | Y | List[str] | 证券代码列表 |
| period | Y | str | 周期 |
| start_time | N | str | 起始时间（日期格式必须是YYYYMMDD等） |
| end_time | N | str | 结束时间（日期格式必须是YYYYMMDD等） |
| count | N | int | 返回数据个数（每只股票） |
| dividend_type | N | str | 复权类型 (opens new window)：none不复权、front前复权、back后复权 |
| fill_data | N | bool | 是否向后填充空缺数据，多只股票一起取k线时使用 |

count小于等于0或者count为空：

1、开始时间与结束时间之间的数据；

2、开始时间无值，从第一根k取到结束时间

3、结束时间无值，从开始时间取到最后一根

4、都没值取全部本地数据

count大于0时：

1、结束时间往前count个数据，起始时间无效

2、结束时间无时从最后一根k线往前取count个数据

### # 返回数据

- 返回dict { field1 : value1, field2 : value2, ... }
- field1, field2, ... ：数据字段
- value1, value2, ... ：pd.DataFrame 数据集，index为stock_list，columns为time_list
- 各字段对应的DataFrame维度相同、索引相同
- 只有dividend_type传入为none时，会返回有效的前复权因子ForwardFactor
- 一次最多返回24000条数据，要获取完整分钟线需要多次分批获取
- 返回的复权因子为权息变动当天的复权因子，一直沿用到下次权息变动
- 开高低收单位为元，成交量单位为量的最小单位(股或张)，成交额单位为万元，对于A股和ETF品种，成交量和成交额不包括盘后固定价格交易的成交。

| 数据 | 默认返回 | 数据类型 | 数据说明 |
|---|---|---|---|
| Date | Y | str | 日期 |
| Time | Y | str | 时间 |
| Open | Y | str | 开盘价 |
| High | Y | str | 最高价 |
| Low | Y | str | 最低价 |
| Close | Y | str | 收盘价 |
| Volume | Y | str | 成交量 |
| Amount | Y | str | 成交额 |
| ForwardFactor | Y | str | 前复权因子，当dividend_type=none时候返回有效值 |
| VolInStock | N | str | 持仓量 |

- 期货数据时Amount为0，非期货数据时VolInStock为0

### # 接口使用

获取688318.SH从2025-12-20到今为止最新一条日K线的不复权数据


```
from tqcenter import tq
tq.initialize(__file__)
df = tq.get_market_data(
field_list=[],
stock_list=['688318.SH'],
start_time='20251220',
end_time='',
count=1,
dividend_type='none',
period='1d',
fill_data=True
)
print(df)
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

10

11

12

13


2

3

4

5

6

7

8

9

10

11

12

13

### # 数据样本

```
{'Close': 688318.SH
2026-09-18 80.52, 'Amount': 688318.SH
2026-09-18 47346.57, 'Volume': 688318.SH
2026-09-18 5962835.0, 'High': 688318.SH
2026-09-18 81.27, 'Open': 688318.SH
2026-09-18 78.6, 'Low': 688318.SH
2026-09-18 77.5, 'VolInStock': 688318.SH
2026-09-18 0.0, 'ForwardFactor': 688318.SH
2026-09-18 1.0}
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
