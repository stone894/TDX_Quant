---
title: "通达信量化平台"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hho7blr2j340/mindoc-1hho9u0cbfe60/portfolio.html
fetched_at: 2026-11-06T06:04:31Z
---

### # `context.portfolio`

当前账户对象

**获取示例策略**

```
def init(context):
context.stock = ["000001.SZ"]
def handle_bar(context, bar_dict):
port = context.portfolio
log.info("available_cash : %s", port.available_cash)
log.info("frozen_cash : %s", port.frozen_cash)
log.info("market_value : %s", port.market_value)
log.info("total_value : %s", port.total_value)
log.info("portfolio_value : %s", port.portfolio_value)
log.info("returns : %s", port.returns)
log.info("pnl : %s", port.pnl)
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

**日志输出**

```
08-17 15:45:38
INFO
提交回测任务
08-17 15:45:38
INFO
回测任务状态：queued
08-17 15:45:38
INFO
回测任务状态：running
08-17 15:45:41
INFO
回测任务状态：succeeded
08-17 15:45:41
INFO
2026-08-17 15:45:39,571 INFO available_cash : 1000000.0
08-17 15:45:41
INFO
2026-08-17 15:45:39,571 INFO frozen_cash : 0.0
08-17 15:45:41
INFO
2026-08-17 15:45:39,571 INFO market_value : 0
08-17 15:45:41
INFO
2026-08-17 15:45:39,571 INFO total_value : 1000000.0
08-17 15:45:41
INFO
2026-08-17 15:45:39,571 INFO portfolio_value : 1000000.0
08-17 15:45:41
INFO
2026-08-17 15:45:39,571 INFO returns : 0.0
08-17 15:45:41
INFO
2026-08-17 15:45:39,571 INFO pnl : 0.0
08-17 15:45:41
INFO
回测任务结束：succeeded
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

25

26

27

28

29

30

31

32

33

34

35

36


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

25

26

27

28

29

30

31

32

33

34

35

36
