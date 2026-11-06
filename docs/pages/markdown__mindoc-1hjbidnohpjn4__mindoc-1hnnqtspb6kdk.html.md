---
title: "获取市场整体情况getglobaldata"
source_url: https://help.tdx.com.cn/quant/docs/markdown/mindoc-1hjbidnohpjn4/mindoc-1hnnqtspb6kdk.html
fetched_at: 2026-11-06T06:02:49Z
---

### # 获取市场整体情况get_global_data

获取沪深京市场的涨跌家数、涨跌停数、主力净额、各主要指数行情与涨跌分布，或获取港股与境外指数、商品、股指期货行情

```
get_global_data(
type: int = 0
) -> dict
```


1

2

3


2

3

#### # 输入参数

| 参数 | 是否必选 | 参数类型 | 参数说明 |
|---|---|---|---|
| type | N | int | 数据类型，0 沪深京市场整体情况（默认），1 港股与全球指数、商品、股指期货行情； |

#### # 返回数据

**市场统计字段（ type=0，30 个）**

| 数据字段 | 数据类型 | 数据说明 |
|---|---|---|
| CurDate | int | 交易日，如 20261023 |
| CurHms | int | 行情时间，HHMMSS |
| UpNum | int | 上涨家数 |
| DownNum | int | 下跌家数 |
| LevelNum | int | 平盘家数 |
| ZtNum | int | 涨停家数 |
| DtNum | int | 跌停家数 |
| ZrztZAF | float | 昨日涨停股今日平均涨幅（%） |
| Zjlx_Main | float | 全市场主力净额（万元） |
| Zjlx_JBuy | float | 全市场主买净额（万元） |
| EverZTNum | int | 炸板家数 |
| LBNum | int | 连板家数 |
| MaxLBDay | int | 最高连板天数 |
| AG_Score1 | int | 全 A 股得分 1 家数 |
| AG_Score2 | int | 全 A 股得分 2 家数 |
| AG_LB | float | 全 A 股量能 |
| MA5_UP | int | 在 5 日均线之上的家数 |
| MA5_Down | int | 在 5 日均线之下的家数 |
| Year_UP | int | 创一年来新高的家数 |
| Year_Down | int | 创一年来新低的家数 |
| HYBK_NowUP | int | 二级研究行业中上涨家数 |
| HYBK_NowDown | int | 二级研究行业中下跌家数 |
| GNBK_NowUP | int | 概念板块中上涨家数 |
| GNBK_NowDown | int | 概念板块中下跌家数 |
| tsum | int | 总家数 |
| tlevel | int | 横盘家数 |
| tup | list[int] | 上涨家数分布，11 个元素，见下文说明 |
| tdown | list[int] | 下跌家数分布，11 个元素，见下文说明 |
| tyx | list[int] | 阳阴线家数，2 个元素：0 阳线、1 阴线 |
| tkb | list[int] | 开板数，2 个元素：0 涨停开板、1 跌停开板 |

**指数行情字段（ type=0，57 个）**

字段名由「指数前缀 + 后缀」拼接，如 `A999999_Now`

为上证指数点位。

| 后缀 | 数据类型 | 数据说明 |
|---|---|---|
| _Now | float | 指数点位 |
| _ZAF | float | 涨幅（%） |
| _Amo | float | 成交额（万元） |
| _LastAmo | float | 昨日成交额（万元） |
| _LastCurAmo | float | 昨日同一时刻成交额（万元） |
| _Zjl | float | 主力净额（万元） |
| _MA5Diff | float | 距离 5 日均线的偏离（%），仅 `A999999` 有 |

| 指数前缀 | 指数名称 | 实际含有的后缀 |
|---|---|---|
| A999999 | 上证指数 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl、MA5Diff |
| A399001 | 深证成指 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl |
| A899050 | 北证50 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl |
| A399006 | 创业板指 | Now、ZAF、Zjl |
| A000688 | 科创50 | Now、ZAF、Zjl |
| A880096 | ETF 指数 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl |
| A000300 | 沪深300 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl |
| A000905 | 中证500 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl |
| A000852 | 中证1000 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl |
| A880823 | 微盘股指数 | Now、ZAF、Amo、LastAmo、LastCurAmo、Zjl |
| A880008 | 全 A 等权指数 | Now、ZAF |


`_Amo`

、`_LastAmo`

、`_LastCurAmo`

、`_Zjl`

的单位均为万元；`_ZAF`

为百分数值，1.71 表示涨幅 1.71%。

`tup`

的 11 个元素按下标对应涨幅区间：0 为涨停，1 为 ≥9%，2 为 [8%, 9%)，…，9 为 [1%, 2%)，10 为 (0%, 1%)。`tdown`

同构：0 为 (0%, -1%)，1 为 [-1%, -2%)，…，9 为 [-9%, ]，10 为跌停。

**全球市场字段（ type=1，53 个）**

| 数据字段 | 数据类型 | 数据说明 |
|---|---|---|
| HK_UpNum | int | 港股上涨家数 |
| HK_DownNum | int | 港股下跌家数 |

香港主板延迟15分钟行情的统计。


其余 51 个字段由「品种代码 + 后缀」拼接，后缀为 `_ZS_Now`

（点位）、`_ZS_ZAF`

（涨幅 %）、`_ZS_Amo`

（成交额，万元）。

| 品种代码 | 名称 | 品种代码 | 名称 |
|---|---|---|---|
| HSI | 恒生指数 | GC00W | COMEX 黄金 |
| HSCEI | 恒生国企指数 | SI00W | COMEX 白银 |
| HSTECH | 恒生科技指数 | HG00W | COMEX 铜 |
| IXIC | 纳斯达克综合指数 | CL00W | NYMEX 原油 |
| HXC | 纳斯达克金龙中国指数 | NG00W | NYMEX 天然气 |
| DAX | 德国 DAX 指数 | IFL8 | 沪深主连 |
| CNY0 | 富时 A50 期指连续 | IHL8 | 上证主连 |
| USDCNH | 美元兑离岸人民币 | ICL8 | 中证主连 |
| IML8 | 中千主连 |


`type=1`

除`HK_UpNum`

、`HK_DownNum`

为整数外，其余 51 个字段的取值全部为字符串（如`"101.06"`

），需要数值计算时自行转换。当日无数据的品种返回`"0.00"`

。

#### # 接口使用

获取沪深京市场整体情况。

```
from tdxaidata import tqs
data = get_global_data(
type=0,
)
print(data["UpNum"], data["DownNum"], data["A999999_Now"])
```


1

2

3

4

5


2

3

4

5

获取港股与全球指数、商品、股指期货行情。

```
from tdxaidata import tqs
data = get_global_data(
type=1,
)
print(data["HSI_ZS_Now"], data["HK_UpNum"])
```


1

2

3

4

5


2

3

4

5

#### # 数据样本

`type=0`

（2026-10-23 15:31 快照，87 个字段全量）

```
{
"CurDate": 20261023,
"CurHms": 153100,
"UpNum": 4526,
"DownNum": 909,
"LevelNum": 137,
"ZtNum": 84,
"DtNum": 7,
"ZrztZAF": 3.38,
"Zjlx_Main": 222972.47,
"Zjlx_JBuy": 10460147.0,
"EverZTNum": 19,
"LBNum": 12,
"MaxLBDay": 6,
"AG_Score1": 832,
"AG_Score2": 1000,
"AG_LB": 1.34,
"MA5_UP": 2842,
"MA5_Down": 2734,
"Year_UP": 80,
"Year_Down": 1204,
"HYBK_NowUP": 115,
"HYBK_NowDown": 13,
"GNBK_NowUP": 267,
"GNBK_NowDown": 2,
"tsum": 5580,
"tlevel": 137,
"tup": [84, 174, 41, 62, 125, 170, 279, 543, 888, 1231, 1013],
"tdown": [443, 204, 70, 53, 44, 21, 26, 17, 10, 21, 7],
"tyx": [5086, 424],
"tkb": [20, 116],
"A999999_Now": 3690.48,
"A999999_ZAF": 0.89,
"A999999_Amo": 89866688.0,
"A999999_LastAmo": 70609568.0,
"A999999_LastCurAmo": 70609552.0,
"A999999_Zjl": -618106.31,
"A999999_MA5Diff": -0.18,
"A399001_Now": 11673.83,
"A399001_ZAF": 2.79,
"A399001_Amo": 105761696.0,
"A399001_LastAmo": 82261800.0,
"A399001_LastCurAmo": 82261800.0,
"A399001_Zjl": 841760.75,
"A899050_Now": 984.9,
"A899050_ZAF": 1.88,
"A899050_Amo": 1693002.13,
"A899050_LastAmo": 1353255.0,
"A899050_LastCurAmo": 1353241.25,
"A899050_Zjl": 0.0,
"A399006_Now": 2684.1,
"A399006_ZAF": 4.16,
"A399006_Zjl": 339283.56,
"A000688_Now": 1345.51,
"A000688_ZAF": 3.04,
"A000688_Zjl": 97871.74,
"A880096_Now": 893.92,
"A880096_ZAF": 1.82,
"A880096_Amo": 69377576.0,
"A880096_LastAmo": 60478660.0,
"A880096_LastCurAmo": 60478628.0,
"A880096_Zjl": 0.0,
"A000300_Now": 4105.33,
"A000300_ZAF": 1.71,
"A000300_Amo": 55820496.0,
"A000300_LastAmo": 43508412.0,
"A000300_LastCurAmo": 43508372.0,
"A000300_Zjl": 16746.15,
"A000905_Now": 6814.44,
"A000905_ZAF": 2.39,
"A000905_Amo": 33694092.0,
"A000905_LastAmo": 25379296.0,
"A000905_LastCurAmo": 25379270.0,
"A000905_Zjl": -103777.8,
"A000852_Now": 6734.55,
"A000852_ZAF": 2.38,
"A000852_Amo": 37768868.0,
"A000852_LastAmo": 29133734.0,
"A000852_LastCurAmo": 29133720.0,
"A000852_Zjl": 37013.49,
"A880823_Now": 6816.03,
"A880823_ZAF": 1.39,
"A880823_Amo": 2531951.0,
"A880823_LastAmo": 1893984.63,
"A880823_LastCurAmo": 1893960.25,
"A880823_Zjl": -4962.62,
"A880008_Now": 1348.11,
"A880008_ZAF": 1.84
}
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

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89


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

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88

89

`type=1`

（同一时刻，53 个字段全量）

```
{
"HK_UpNum": 1514,
"HK_DownNum": 870,
"HSI_ZS_Now": "23093.37",
"HSI_ZS_ZAF": "0.69",
"HSI_ZS_Amo": "24978358.00",
"HSCEI_ZS_Now": "7756.25",
"HSCEI_ZS_ZAF": "0.87",
"HSCEI_ZS_Amo": "8714180.00",
"HSTECH_ZS_Now": "3871.66",
"HSTECH_ZS_ZAF": "2.05",
"HSTECH_ZS_Amo": "6927877.50",
"IXIC_ZS_Now": "0.00",
"IXIC_ZS_ZAF": "0.00",
"IXIC_ZS_Amo": "0.00",
"HXC_ZS_Now": "0.00",
"HXC_ZS_ZAF": "0.00",
"HXC_ZS_Amo": "0.00",
"DAX_ZS_Now": "25132.23",
"DAX_ZS_ZAF": "-0.94",
"DAX_ZS_Amo": "0.00",
"CNY0_ZS_Now": "13005.00",
"CNY0_ZS_ZAF": "-0.26",
"CNY0_ZS_Amo": "0.00",
"USDCNH_ZS_Now": "6.67",
"USDCNH_ZS_ZAF": "-0.06",
"USDCNH_ZS_Amo": "0.00",
"GC00W_ZS_Now": "4036.70",
"GC00W_ZS_ZAF": "-0.58",
"GC00W_ZS_Amo": "27026.10",
"SI00W_ZS_Now": "53.84",
"SI00W_ZS_ZAF": "-2.12",
"SI00W_ZS_Amo": "130.31",
"HG00W_ZS_Now": "6.45",
"HG00W_ZS_ZAF": "0.74",
"HG00W_ZS_Amo": "8.56",
"CL00W_ZS_Now": "101.05",
"CL00W_ZS_ZAF": "1.59",
"CL00W_ZS_Amo": "642.37",
"NG00W_ZS_Now": "3.13",
"NG00W_ZS_ZAF": "-0.98",
"NG00W_ZS_Amo": "3.98",
"IFL8_ZS_Now": "4247.60",
"IFL8_ZS_ZAF": "0.27",
"IFL8_ZS_Amo": "9075473.00",
"IHL8_ZS_Now": "2764.20",
"IHL8_ZS_ZAF": "0.24",
"IHL8_ZS_Amo": "2927109.75",
"ICL8_ZS_Now": "7111.60",
"ICL8_ZS_ZAF": "0.24",
"ICL8_ZS_Amo": "16101745.00",
"IML8_ZS_Now": "7018.60",
"IML8_ZS_ZAF": "0.40",
"IML8_ZS_Amo": "27979970.00"
}
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
