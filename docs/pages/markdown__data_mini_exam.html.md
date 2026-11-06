---
title: "1、取中证2000指数的最近1000只日K线"
source_url: https://help.tdx.com.cn/quant/docs/markdown/data_mini_exam.html
fetched_at: 2026-11-06T06:02:39Z
---

### # 1、取中证2000指数的最近1000只日K线

用tqserver(即tdxaidata)方法

```
df = tqs.get_market_data(
field_list=[],
stock_list=['932000.CSI'],
start_time='',
end_time='',
count=1000,
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

### # 2、列出所有的宽基ETF

```
block_stocks = tqs.get_stock_list_in_sector('880698.SH')
print(block_stocks)
```


1

2


2

### # 3、美股热门ETF按涨幅排名，取前80只，同时返回量比和总市值

```
rank = tqs.get_domain_sorthq(
domain=13006,
colname="ZAF",
sorttype=1,
wantcol=["fLianB","Zsz"],
startxh=0,
wantnum=80,
data_source=1,
)
print(rank)
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


2

3

4

5

6

7

8

9

10

### # 4、科创板中，按这个公式源码进行日线级选股，最近2天满足条件

通达信公式源码：连涨四天

UP4:UPNDAY(CLOSE,4);

以下为主要调用代码，用tqserver(即tdxaidata)方法

```
result = tqs.formula_by_src(
formula_src='UP4:UPNDAY(CLOSE,4);',
domain=8,
stock_period='1d',
count=100,
return_count=2,
return_date=True,
dividend_type=1,
formula_type=1
)
print(result)
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

### # 5、获取880、881系列任意板块成份股20日涨幅前5

前缀"880"板块成份股 20+880代码的后3位,前缀"881"板块成份股 21+881代码的后3位

```
# 881004 研究行业三级 炼焦煤
from tdxaidata import tqs
rank = tqs.get_domain_sorthq(
domain=21004,
colname="ZAFPre20",
sorttype=1,
wantcol=["Amount", "fHSL","ZTDateNum","ZAFPre20"],
startxh=0,
wantnum=5,
)
print(rank)
# 880302 普通细分行业 煤炭开采
from tdxaidata import tqs
rank = tqs.get_domain_sorthq(
domain=20302,
colname="ZAFPre20",
sorttype=1,
wantcol=["Amount", "fHSL","ZTDateNum","ZAFPre20"],
startxh=0,
wantnum=5,
)
print(rank)
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

14

15

16

17

18

19

20

21

22

23

24


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

14

15

16

17

18

19

20

21

22

23

24

### # 6、获取香港HK、美股US开头系列板块成份股3日涨幅前5

前缀"HK"板块成份股 2+HK代码的后4位 前缀"US"板块成份股 3+US代码的后4位 data_source=1

```
from tdxaidata import tqs
rank = tqs.get_domain_sorthq(
domain=30302,
colname="ZAFPre3",
sorttype=1,
wantcol=["Amount", "fHSL","ZAFPre3"],
startxh=0,
wantnum=5,
data_source=1
)
print(rank)
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
