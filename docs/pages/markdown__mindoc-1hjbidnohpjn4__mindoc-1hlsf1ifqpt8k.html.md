---
title: "取消实时行情订阅 unsubscribe"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hjbidnohpjn4/mindoc-1hlsf1ifqpt8k.html
fetched_at: 2026-11-06T06:03:24Z
---

### # 取消实时行情订阅 unsubscribe

取消指定股票的实时行情订阅

```
unsubscribe() -> None
```


1


#### # 返回数据

`unsubscribe`

无返回数据，返回值为：

```
None
```


1


#### # 接口使用

订阅两只股票，运行 30 秒后取消订阅。


```
import time
from tdxaidata import tqs
stock_list = [
"000001.SZ",
"600000.SH",
]
subscribed = False
try:
tqs.subscribe(
stock_list=stock_list,
callback=lambda data: print(data),
)
subscribed = True
print("当前订阅列表：")
print(tqs.get_subscribe_hq_stock_list())
time.sleep(30)
finally:
if subscribed:
tqs.unsubscribe()
print("已取消订阅")
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

#### # 注意事项

- 取消订阅后，不再接收对应行情回调。
- 当前实现为全局取消订阅接口，不是只取消传入的部分股票。
- 如果需要保留其他股票的订阅，应在取消后重新调用
`subscribe`

。 - 建议在
`finally`

代码块中调用`unsubscribe`

，确保程序异常退出时也能释放订阅。
