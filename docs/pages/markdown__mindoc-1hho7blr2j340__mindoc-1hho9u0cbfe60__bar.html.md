---
title: "通达信量化平台"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hho7blr2j340/mindoc-1hho9u0cbfe60/bar.html
fetched_at: 2026-11-06T06:04:34Z
---

### # `context.bar`

Bar对象

**获取示例策略**

```
def init(context):
context.stock = ["000001.SZ"]
def handle_bar(context, bar_dict):
bar = bar_dict["000001.SZ"]
log.info("symbol : %s", bar.symbol)
log.info("datetime : %s", bar.datetime)
log.info("open : %s", bar.open)
log.info("high : %s", bar.high)
log.info("low : %s", bar.low)
log.info("close : %s", bar.close)
log.info("volume : %s", bar.volume)
log.info("amount : %s", bar.amount)
log.info("money : %s", bar.money)
log.info("high_limit : %s", bar.high_limit)
log.info("low_limit : %s", bar.low_limit)
log.info("paused : %s", bar.paused)
log.info("is_paused : %s", bar.is_paused)
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

**日志输出**

```
08-17 15:51:31
INFO
提交回测任务
08-17 15:51:31
INFO
回测任务状态：queued
08-17 15:51:31
INFO
回测任务状态：running
08-17 15:51:34
INFO
回测任务状态：succeeded
08-17 15:51:34
INFO
2026-08-17 15:51:32,208 INFO symbol : 000001.SZ
08-17 15:51:34
INFO
2026-08-17 15:51:32,208 INFO datetime : 2026-08-12 00:00:00
08-17 15:51:34
INFO
2026-08-17 15:51:32,208 INFO open : 11.260000228881836
08-17 15:51:34
INFO
2026-08-17 15:51:32,208 INFO high : 11.289999961853027
08-17 15:51:34
INFO
2026-08-17 15:51:32,208 INFO low : 11.199999809265137
08-17 15:51:34
INFO
2026-08-17 15:51:32,208 INFO close : 11.25
08-17 15:51:34
INFO
2026-08-17 15:51:32,209 INFO volume : 63295024.0
08-17 15:51:34
INFO
2026-08-17 15:51:32,209 INFO amount : 711612864.0
08-17 15:51:34
INFO
2026-08-17 15:51:32,209 INFO money : 711612864.0
08-17 15:51:34
INFO
2026-08-17 15:51:32,209 INFO high_limit : None
08-17 15:51:34
INFO
2026-08-17 15:51:32,209 INFO low_limit : None
08-17 15:51:34
INFO
2026-08-17 15:51:32,209 INFO paused : False
08-17 15:51:34
INFO
2026-08-17 15:51:32,209 INFO is_paused : False
08-17 15:51:34
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
