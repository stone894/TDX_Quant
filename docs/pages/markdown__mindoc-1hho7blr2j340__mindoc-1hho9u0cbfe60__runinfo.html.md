---
title: "通达信量化平台"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hho7blr2j340/mindoc-1hho9u0cbfe60/runinfo.html
fetched_at: 2026-11-06T06:04:35Z
---

### # `context.run_info`

回测运行信息对象

**获取示例策略**

```
def init(context):
context.stock = ["000001.SZ"]
def handle_bar(context, bar_dict):
ri = context.run_info
log.info("start_date : %s", ri.start_date)
log.info("end_date : %s", ri.end_date)
log.info("benchmark : %s", ri.benchmark)
log.info("frequency : %s", ri.frequency)
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

**日志输出**

```
08-17 15:52:36
INFO
提交回测任务
08-17 15:52:36
INFO
回测任务状态：queued
08-17 15:52:36
INFO
回测任务状态：running
08-17 15:52:39
INFO
回测任务状态：succeeded
08-17 15:52:39
INFO
2026-08-17 15:52:37,746 INFO start_date : 2026-08-12
08-17 15:52:39
INFO
2026-08-17 15:52:37,746 INFO end_date : 2026-08-12
08-17 15:52:39
INFO
2026-08-17 15:52:37,746 INFO benchmark : None
08-17 15:52:39
INFO
2026-08-17 15:52:37,746 INFO frequency : 1d
08-17 15:52:39
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
