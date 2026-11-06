# 通达信全市场概念板块 RSLINE 排序｜完整代码

## 目标

调用自定义通达信技术指标公式 `RSLINE`，分别对通达信全市场**概念板块**和**行业板块**进行批量计算，取每个板块最新交易日的 `RSLINE` 值并从高到低排序，分别导出 CSV。

官方接口要点：

- `get_stock_list(market="12", list_type=1)`：获取概念板块代码和名称。
- `get_stock_list(market="11", list_type=1)`：获取通达信缺省行业板块代码和名称。
- `formula_process_mul_zb()`：批量调用技术指标公式。
- 为计算“最新值相对5个交易日前”的变化，设置 `return_count=6`；6个观测点之间包含5个交易日间隔。
- 板块指数一般使用 `dividend_type=0`，即不复权。

---

## 原始 RSLINE 公式

```
INDEXC1:="000300$CLOSE", NODRAW;

SETIDX:=TDXDLL1(1, 0, 0, INDEXC1);

{第三步：计算RS线（偏移系数=25）}
RSLINE:TDXDLL1(2, 25, 0, CLOSE), COLORRED, LINETHICK2;

{第四步：RS线的均线；当前DLL参数为21}
RSMA50:TDXDLL1(3, 21, 0, RSLINE), COLOR00FF00;

{第五步：检测40周期新高}
NEWHIGH:=TDXDLL1(4, 40, 0, RSLINE);
DRAWICON(NEWHIGH=1, RSLINE*1.07, 70);

GDAREA:=RSLINE>RSMA50;
BDAREA:=RSLINE<RSMA50;

GDSTICK:STICKLINE(GDAREA, RSMA50, RSLINE, 1, 0), COLORRED;
BDSTICK:STICKLINE(BDAREA, RSMA50, RSLINE, 1, 0), COLORGREEN;

RSL:=RSLINE-RSMA50;
```

<aside>
⚠️

公式注释写的是“50日SMA”，但实际传给 DLL 的参数是 `21`：`TDXDLL1(3, 21, 0, RSLINE)`。以下代码不改变公式，只读取其输出。如果你确实需要50日线，应先确认DLL第3号功能的参数定义，再决定是否将21改为50。

</aside>

### 输出字段说明

`RSLINE:` 和 `RSMA50:` 使用冒号，是可返回的公式输出线。`RSL:=...` 使用 `:=`，属于内部变量，通常不会出现在批量接口结果中。本方案直接读取返回结果里的 `RSLINE`。

---

## 完整 Python 代码

```python
from pathlib import Path
from datetime import datetime
import time
import pandas as pd
from tqcenter import tq

# ============================================================
# 配置区
# ============================================================

FORMULA_NAME = "RSLINE"       # 通达信公式管理器中的公式名称
FORMULA_ARG = ""              # 当前公式没有外部参数
STOCK_PERIOD = "1d"           # 日线
KLINE_COUNT = 500              # 给DLL及长周期计算预留足够历史数据
RETURN_COUNT = 6               # 最新值+前5个交易日，共6个观测点
XSFLAG = 8                     # 最大返回8位小数
DIVIDEND_TYPE = 0              # 板块指数不复权
BATCH_SIZE = 200               # 分批计算，降低单次返回压力
SLEEP_SECONDS = 0.2            # 批次间短暂停顿

OUTPUT_DIR = Path(__file__).resolve().parent / "output"
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

# ============================================================
# 工具函数
# ============================================================

def split_batches(items, batch_size):
    """把列表切成若干批次。"""
    for start in range(0, len(items), batch_size):
        yield items[start:start + batch_size]

def get_latest_value(output_data, field_name):
    """
    从单个板块的公式返回结果中提取指定输出线的最新值。

    return_date=True时，常见结构为：
    {
        "RSLINE": [
            {"Date": "20260807", "Value": "123.456"}
        ]
    }
    """
    if not isinstance(output_data, dict):
        return None, None

    series = output_data.get(field_name)

    if not series or not isinstance(series, list):
        return None, None

    # 从后往前寻找最后一个有效值
    for item in reversed(series):
        if isinstance(item, dict):
            trade_date = item.get("Date")
            raw_value = item.get("Value")
        else:
            trade_date = None
            raw_value = item

        if raw_value in (None, "", "None"):
            continue

        try:
            value = float(raw_value)
            return trade_date, value
        except (TypeError, ValueError):
            continue

    return None, None

def get_recent_numeric_values(output_data, field_name):
    """按接口返回顺序提取某条公式输出线的有效数值。"""
    if not isinstance(output_data, dict):
        return []

    series = output_data.get(field_name)
    if not series or not isinstance(series, list):
        return []

    values = []
    for item in series:
        if isinstance(item, dict):
            trade_date = item.get("Date")
            raw_value = item.get("Value")
        else:
            trade_date = None
            raw_value = item

        if raw_value in (None, "", "None"):
            continue

        try:
            values.append({
                "Date": trade_date,
                "Value": float(raw_value)
            })
        except (TypeError, ValueError):
            continue

    return values

def normalize_sector_list(raw_list):
    """兼容list_type=1可能返回的代码、名称字段格式。"""
    concepts = []

    for item in raw_list:
        if isinstance(item, dict):
            code = item.get("Code") or item.get("code")
            name = item.get("Name") or item.get("name") or ""
        elif isinstance(item, str):
            code = item
            name = ""
        else:
            continue

        if code:
            concepts.append({"Code": str(code), "Name": str(name)})

    # 按代码去重
    unique = {}
    for item in concepts:
        unique[item["Code"]] = item

    return list(unique.values())

# ============================================================
# 主程序
# ============================================================

def run_sector_ranking(market_code, sector_label):
    """计算指定板块分类的RSLINE指标并分别导出排名文件。"""

    # market_code="12"：概念板块
    # market_code="11"：通达信缺省行业板块
    raw_sectors = tq.get_stock_list(
        market=market_code,
        list_type=1
    )

    sectors = normalize_sector_list(raw_sectors)

    if not sectors:
        raise RuntimeError(
            f"未取得{sector_label}列表，请检查通达信客户端和TQ连接。"
        )

    code_name_map = {
        item["Code"]: item["Name"]
        for item in sectors
    }
    concept_codes = list(code_name_map.keys())

    print(f"\n共取得 {len(concept_codes)} 个{sector_label}")
    print(f"开始批量计算{sector_label}RSLINE……")

    rows = []
    failed_rows = []
    total_batches = (len(concept_codes) + BATCH_SIZE - 1) // BATCH_SIZE

    # 3. 分批调用自定义指标公式
    for batch_no, code_batch in enumerate(
        split_batches(concept_codes, BATCH_SIZE),
        start=1
    ):
        print(
            f"正在处理第 {batch_no}/{total_batches} 批，"
            f"本批 {len(code_batch)} 个板块"
        )

        try:
            result = tq.formula_process_mul_zb(
                formula_name=FORMULA_NAME,
                formula_arg=FORMULA_ARG,
                xsflag=XSFLAG,
                return_count=RETURN_COUNT,
                return_date=True,
                stock_list=code_batch,
                stock_period=STOCK_PERIOD,
                count=KLINE_COUNT,
                dividend_type=DIVIDEND_TYPE
            )
        except Exception as exc:
            # 整批失败时记录，继续下一批
            for code in code_batch:
                failed_rows.append({
                    "板块代码": code,
                    "板块名称": code_name_map.get(code, ""),
                    "失败原因": f"批量调用异常：{exc}"
                })
            continue

        if not isinstance(result, dict):
            for code in code_batch:
                failed_rows.append({
                    "板块代码": code,
                    "板块名称": code_name_map.get(code, ""),
                    "失败原因": "接口返回值不是字典"
                })
            continue

        error_id = str(result.get("ErrorId", "0"))
        if error_id != "0":
            print(f"第 {batch_no} 批接口ErrorId={error_id}")

        for code in code_batch:
            output_data = result.get(code)

            if output_data is None:
                failed_rows.append({
                    "板块代码": code,
                    "板块名称": code_name_map.get(code, ""),
                    "失败原因": "返回结果中没有该板块"
                })
                continue

            trade_date, rsline_value = get_latest_value(
                output_data,
                "RSLINE"
            )

            if rsline_value is None:
                fields = (
                    list(output_data.keys())
                    if isinstance(output_data, dict)
                    else []
                )
                failed_rows.append({
                    "板块代码": code,
                    "板块名称": code_name_map.get(code, ""),
                    "失败原因": f"未取得RSLINE；实际字段={fields}"
                })
                continue

            # RSMA50是公式的第二条输出线，顺便读取，便于观察强弱状态
            _, rsma50_value = get_latest_value(
                output_data,
                "RSMA50"
            )

            # 标准化乖离率：消除不同RSLINE数值水平带来的尺度影响
            # 正值表示RSLINE在均线上方，负值表示在均线下方
            rs_deviation_pct = None
            if rsma50_value not in (None, 0):
                rs_deviation_pct = (
                    rsline_value / rsma50_value - 1
                ) * 100

            # 取最新值和5个交易日前的值。
            # 计算5日变化需要6个观测点，因此RETURN_COUNT必须至少为6。
            rsline_history = get_recent_numeric_values(
                output_data,
                "RSLINE"
            )
            rsma_history = get_recent_numeric_values(
                output_data,
                "RSMA50"
            )

            # 检测RSLINE相对RSMA50由负转正：
            # 前一日差值小于0，最新差值大于0时返回True。
            # 等价于RSLINE从RSMA50下方上穿到RSMA50上方。
            rsline_turn_up = False

            if len(rsline_history) >= 2 and len(rsma_history) >= 2:
                previous_spread = (
                    rsline_history[-2]["Value"]
                    - rsma_history[-2]["Value"]
                )
                latest_spread = (
                    rsline_history[-1]["Value"]
                    - rsma_history[-1]["Value"]
                )

                rsline_turn_up = (
                    previous_spread < 0
                    and latest_spread > 0
                )

            rsline_5d_change_pct = None
            if len(rsline_history) >= 6:
                rsline_5d_ago = rsline_history[-6]["Value"]
                rsline_latest = rsline_history[-1]["Value"]
                if rsline_5d_ago != 0:
                    rsline_5d_change_pct = (
                        rsline_latest / rsline_5d_ago - 1
                    ) * 100

            # RSMA五日斜率率：先算5日累计变化率，再除以5，
            # 表示这5个交易日内的日均复合近似斜率率。
            rsma_5d_slope_pct = None
            if len(rsma_history) >= 6:
                rsma_5d_ago = rsma_history[-6]["Value"]
                rsma_latest = rsma_history[-1]["Value"]
                if rsma_5d_ago != 0:
                    rsma_5d_slope_pct = (
                        (rsma_latest / rsma_5d_ago - 1) * 100
                    ) / 5

            rows.append({
                "板块代码": code,
                "板块名称": code_name_map.get(code, ""),
                "交易日期": trade_date,
                "RSLINE": rsline_value,
                "RSMA50": rsma50_value,
                "RS乖离率%": rs_deviation_pct,
                "RSLINE上穿RSMA50": rsline_turn_up,
                "RSLINE五日变化率%": rsline_5d_change_pct,
                "RSMA五日斜率率%/日": rsma_5d_slope_pct,
                "动量趋势同步向上": (
                    rsline_5d_change_pct is not None
                    and rsma_5d_slope_pct is not None
                    and rsline_5d_change_pct > 0
                    and rsma_5d_slope_pct > 0
                ),
                "位于均线上方": (
                    rsline_value > rsma50_value
                    if rsma50_value is not None
                    else None
                )
            })

        time.sleep(SLEEP_SECONDS)

    # 4. 生成结果表
    df = pd.DataFrame(rows)

    if df.empty:
        raise RuntimeError(
            "没有取得任何有效RSLINE值。请检查公式名称、DLL加载、"
            "公式输出字段和盘后K线数据。"
        )

    # 使用所有返回结果中的最新日期作为目标日期，避免停更板块混入排名
    available_dates = df["交易日期"].dropna()
    target_date = available_dates.max() if not available_dates.empty else None

    if target_date:
        stale_df = df[df["交易日期"] != target_date].copy()
        rank_df = df[df["交易日期"] == target_date].copy()
    else:
        stale_df = pd.DataFrame(columns=df.columns)
        rank_df = df.copy()

    # RSLINE从高到低排名
    rank_df = rank_df.sort_values(
        by=["RSLINE", "板块代码"],
        ascending=[False, True],
        na_position="last"
    ).reset_index(drop=True)

    rank_df.insert(0, "RSLINE排名", range(1, len(rank_df) + 1))

    # 对标准化后的RS乖离率进行独立排名
    # 乖离率越大排名越靠前，但排名过高也可能意味着短期过热
    if "RS乖离率%" in rank_df.columns:
        rank_df["乖离率排名"] = rank_df["RS乖离率%"].rank(
            method="min",
            ascending=False,
            na_option="bottom"
        ).astype("Int64")

        # 全市场乖离率最高5%标记为短期过热观察区
        valid_deviation = rank_df["RS乖离率%"].dropna()
        if not valid_deviation.empty:
            overheat_line = valid_deviation.quantile(0.95)
            rank_df["短期过热"] = (
                rank_df["RS乖离率%"] >= overheat_line
            )
        else:
            rank_df["短期过热"] = False

    # RSLINE五日变化率排名：观察相对强度的短期动量
    rank_df["五日动量排名"] = rank_df["RSLINE五日变化率%"].rank(
        method="min",
        ascending=False,
        na_option="bottom"
    ).astype("Int64")

    # RSMA五日斜率率排名：观察中短期相对强度趋势的上升速度
    rank_df["RSMA斜率排名"] = rank_df["RSMA五日斜率率%/日"].rank(
        method="min",
        ascending=False,
        na_option="bottom"
    ).astype("Int64")

    # 5. 导出
    stamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    main_file = OUTPUT_DIR / f"{sector_label}_RSLINE排名_{stamp}.csv"
    rank_df.to_csv(main_file, index=False, encoding="utf-8-sig")

    if not stale_df.empty:
        stale_file = OUTPUT_DIR / f"{sector_label}_非最新日期_{stamp}.csv"
        stale_df.to_csv(stale_file, index=False, encoding="utf-8-sig")
    else:
        stale_file = None

    if failed_rows:
        failed_df = pd.DataFrame(failed_rows)
        failed_file = OUTPUT_DIR / f"{sector_label}_RSLINE失败记录_{stamp}.csv"
        failed_df.to_csv(failed_file, index=False, encoding="utf-8-sig")
    else:
        failed_file = None

    # 6. 屏幕输出
    print(f"\n{sector_label}计算完成")
    print(f"目标交易日：{target_date}")
    print(f"有效排名数量：{len(rank_df)}")
    print(f"非最新日期数量：{len(stale_df)}")
    print(f"失败数量：{len(failed_rows)}")
    print(
        f"RSLINE上穿RSMA50数量："
        f"{int(rank_df['RSLINE上穿RSMA50'].sum())}"
    )
    print(f"主结果文件：{main_file}")

    if stale_file:
        print(f"非最新日期文件：{stale_file}")

    if failed_file:
        print(f"失败记录文件：{failed_file}")

    display_columns = [
        "RSLINE排名",
        "板块代码",
        "板块名称",
        "交易日期",
        "RSLINE",
        "RSMA50",
        "RS乖离率%",
        "RSLINE上穿RSMA50",
        "RSLINE五日变化率%",
        "RSMA五日斜率率%/日",
        "动量趋势同步向上",
        "位于均线上方",
        "乖离率排名",
        "五日动量排名",
        "RSMA斜率排名",
        "短期过热"
    ]

    print("\nRSLINE排名前50：")
    print(
        rank_df[display_columns]
        .head(50)
        .to_string(index=False)
    )

    return rank_df

def main():
    # 只初始化一次TQ，然后依次计算概念板块和行业板块
    tq.initialize(__file__)

    concept_rank_df = run_sector_ranking(
        market_code="12",
        sector_label="概念板块"
    )

    industry_rank_df = run_sector_ranking(
        market_code="11",
        sector_label="行业板块"
    )

    return concept_rank_df, industry_rank_df

if __name__ == "__main__":
    main()
```

---

## 运行前检查

1. 启动并登录支持 TQ 策略功能的通达信客户端。
2. 确认公式管理器中的技术指标公式名称确实为 `RSLINE`。
3. 确认 `TDXDLL1` 已正确安装、加载，且量化进程能调用该 DLL。
4. 下载完整的板块指数日线盘后数据；数据不足会导致结果与客户端不一致。
5. 确认沪深300基准表达式 `"000300$CLOSE"` 在批量板块计算环境中能够正常取得数据。
6. 首次运行可分别把概念板块和行业板块代码列表切成前5个验证，再运行完整板块池。
7. 正常运行结束后，`output` 目录会同时生成 `概念板块_RSLINE排名_时间.csv` 和 `行业板块_RSLINE排名_时间.csv`。

## 首次小样本验证

在正式批量运行前，可以临时把：

```python
concept_codes = list(code_name_map.keys())
```

改为：

```python
concept_codes = list(code_name_map.keys())[:5]
```

核对以下内容：

- 返回结果中是否存在 `RSLINE` 字段；
- Python最后一个值是否与通达信客户端同日数值一致；
- `ErrorId` 是否为 `0`；
- 各板块返回的交易日期是否一致；
- DLL在批量运行时是否会在不同板块之间正确重置基准序列。

## 输出文件

程序每次运行会分别输出：

- `概念板块_RSLINE排名_时间.csv`：`market="12"`的概念板块排名。
- `行业板块_RSLINE排名_时间.csv`：`market="11"`的通达信缺省行业板块排名。
- 如存在数据日期落后或计算失败，还会按板块类别分别输出“非最新日期”和“失败记录”文件。

如需使用研究行业口径，可把行业调用中的 `market_code` 改为：`"16"`（研究行业一级）、`"17"`（研究行业二级）或 `"18"`（研究行业三级）。

## 结果解读

- `RSLINE排名`：在各自板块池内部，按最新RSLINE由高到低排列；概念板块与行业板块不会混排。
- `RS乖离率%`：标准化乖离率，计算公式为 `(RSLINE ÷ RSMA50 - 1) × 100%`。
- `RSLINE上穿RSMA50`：当前一日的 `RSLINE-RSMA50 < 0`，且最新一日的 `RSLINE-RSMA50 > 0` 时显示 `True`。这表示RSLINE刚从RSMA50下方上穿到上方。严格按“负转正”处理，前一日或最新一日差值恰好等于0时不会触发。
- `RSLINE五日变化率%`：`(最新RSLINE ÷ 5个交易日前RSLINE - 1) × 100%`，衡量相对强度的5日动量。
- `RSMA五日斜率率%/日`：`[(最新RSMA50 ÷ 5个交易日前RSMA50 - 1) × 100%] ÷ 5`，衡量RS均线在最近5个交易日内的日均变化速度。
- `动量趋势同步向上`：RSLINE五日变化率和RSMA五日斜率率同时大于0。
- `位于均线上方`：对应公式中的 `GDAREA` 条件。
- `乖离率排名`：按标准化乖离率从高到低排名，用于观察相对强度的近期加速程度。
- `五日动量排名`：按RSLINE五日变化率从高到低排名。
- `RSMA斜率排名`：按RSMA五日斜率率从高到低排名。
- `短期过热`：乖离率位于全市场最高5%的板块，仅作为风险提示，不代表必然见顶。

<aside>
📌

`RSLINE上穿RSMA50=True`表示最新交易日刚刚完成由负乖离向正乖离的穿越，是转强触发信号，但不等于中期趋势已经反转。更可靠的确认还应结合五日变化率转正、RSMA斜率转正及RSLINE排名持续前移。RSLINE高低的最终含义仍取决于TDXDLL1第2号功能的内部算法，排序前应先用少量板块与客户端结果交叉验证。

</aside>

[改进版｜RSLINE 多周期趋势排序 Python 代码](https://app.notion.com/p/RSLINE-Python-8768bd53d5e2499d900935a930b628bc?pvs=21)