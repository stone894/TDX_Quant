---
title: "Trade对象"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hho7blr2j340/mindoc-1hho9u0cbfe60/trade.html
fetched_at: 2026-11-06T06:04:37Z
---

### # Trade对象

`注意`


Order：支持单个拿

下单返回order_id，用 get_order(order_id)，可以精准拿到某一个指定订单对象，逐个访问。

Trade：没有get_trade(trade_id)这种单条接口，只能批量

只能调用 get_trades() 返回全部成交列表，再自己做过滤：按order_id、时间、标的筛选出你想要的那一笔 / 多笔成交。

**获取示例策略**

```
def init(context):
context.stock = ["000001.SZ","000006.SZ"]
context.traded = False
def handle_bar(context, bar_dict):
if not context.traded:
order_target_value("000001.SZ",10000)
order_target_value("000006.SZ",10000)
context.traded = True
trade_list = get_trades()
for t in trade_list:
try:
log.info("trade_id : %s", t.trade_id)
except Exception as e:
log.info("trade_id : ERR=%s", str(e))
try:
log.info("order_id : %s", t.order_id)
except Exception as e:
log.info("order_id : ERR=%s", str(e))
try:
log.info("symbol : %s", t.symbol)
except Exception as e:
log.info("symbol : ERR=%s", str(e))
try:
log.info("amount : %s", t.amount)
except Exception as e:
log.info("amount : ERR=%s", str(e))
try:
log.info("price : %s", t.price)
except Exception as e:
log.info("price : ERR=%s", str(e))
try:
log.info("side : %s", t.side)
except Exception as e:
log.info("side : ERR=%s", str(e))
try:
log.info("datetime : %s", t.datetime)
except Exception as e:
log.info("datetime : ERR=%s", str(e))
try:
log.info("cost : %s", t.cost)
except Exception as e:
log.info("cost : ERR=%s", str(e))
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

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51


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

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

**日志输出**

```
08-17 15:54:51
INFO
提交回测任务
08-17 15:54:51
INFO
回测任务状态：queued
08-17 15:54:51
INFO
回测任务状态：running
08-17 15:54:54
INFO
回测任务状态：succeeded
08-17 15:54:54
INFO
2026-08-17 15:54:52,311 INFO trade_id : T00000001
08-17 15:54:54
INFO
2026-08-17 15:54:52,311 INFO order_id : L00000001
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO symbol : 000001.SZ
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO amount : 800.0
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO price : 11.260000228881836
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO side : buy
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO datetime : 2026-08-12 00:00:00
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO cost : 5.09
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO trade_id : T00000002
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO order_id : L00000002
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO symbol : 000006.SZ
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO amount : 1400.0
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO price : 6.809999942779541
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO side : buy
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO datetime : 2026-08-12 00:00:00
08-17 15:54:54
INFO
2026-08-17 15:54:52,312 INFO cost : 5.1
08-17 15:54:54
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

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63


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

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63
