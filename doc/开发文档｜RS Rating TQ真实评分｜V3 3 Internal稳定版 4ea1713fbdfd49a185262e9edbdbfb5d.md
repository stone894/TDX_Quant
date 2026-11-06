# 开发文档｜RS Rating TQ真实评分｜V3.3 Internal稳定版

<aside>
✅

稳定基线：V3.3 Internal（TQ内部运行版）。2026-09-02用户确认“功能完全正常，界面已经可以显示了”。2026-11-06整理归档；不修改已验收代码。此记录不代表已完成全市场性能验收或历史无偏回测。

</aside>

## 1. 稳定版源码与恢复包

[rs_rating_tq_v3.3_[internal.zip](http://internal.zip)｜最后确认可正常运行的完整源码](rs_rating_tq_v3.3_internal.zip)

rs_rating_tq_v3.3_[internal.zip](http://internal.zip)｜最后确认可正常运行的完整源码

本附件保留当时交付的原始ZIP，不以外部运行实验版V3.4替代。解压后包含：

- `rs_rating_tq.py`：行情获取、参数配置、持久化与TQ推送主程序。
- `rs_core.py`：Raw Score、有效样本、横截面排名与阈值计算。
- `RSRATING_EOD.tdx`：通达信日线副图公式。
- `test_rs_core.py`：不依赖TQ的合成数据测试。
- `README.md`：运行说明。

ZIP的SHA-256：

```
c7df051634989d48cf1cd78dd457eab4435ee6f1bd36f71aa0d3236a71ef901f
```

关键文件的SHA-256：

```
rs_rating_tq.py
b2bee14493fc5fff10ab3ff3b0863a2c540a4a69114c13a3a98d36b2c7be2edc

rs_core.py
14ada785104ac537fb558ad855195299b56325f26e4241b9cf666354198437f4

RSRATING_EOD.tdx
8fc7065d7e1bdc9c79b964b1d7a2a0c78d0b5621ac7a87cb287809da6fabeed5
```

归档时重新执行Python语法检查、现有合成数据测试和ZIP完整性检查，均通过。没有在归档环境重新连接用户Windows通达信；客户端显示验收依据为2026-09-02用户反馈。

## 稳定版完整源代码（可直接复制）

以下代码与本页归档的V3.3 Internal稳定包一致，未改动算法及默认参数。分别保存为下列文件；两个Python模块必须放在同一目录。运行主程序时使用已验证的TQ内部入口，不传命令行参数。默认30只股票、400根日线仅用于联调，评分是测试样本内排名。

### rs_core.py｜计算核心

```python
from __future__ import annotations

import numpy as np
import pandas as pd

LOOKBACKS = (63, 126, 189, 252)
WEIGHTS = (0.40, 0.20, 0.20, 0.20)

def calculate_raw_score(close: pd.DataFrame) -> pd.DataFrame:
    """计算加权价格表现原始分数；结果以100为大致中枢。"""
    close = close.astype(float).sort_index()
    score = pd.DataFrame(0.0, index=close.index, columns=close.columns)
    valid = pd.DataFrame(True, index=close.index, columns=close.columns)

    for n, weight in zip(LOOKBACKS, WEIGHTS):
        ratio = close / close.shift(n)
        score = score + weight * ratio
        valid &= ratio.notna() & np.isfinite(ratio) & (ratio > 0)

    return (score * 100.0).where(valid)

def calculate_valid_mask(
    observed_close: pd.DataFrame,
    min_coverage: float = 0.95,
    recent_window: int = 20,
    min_recent_trading_days: int = 15,
) -> pd.DataFrame:
    """生成逐日样本有效性掩码，不把填充价格当成真实成交。"""
    observed = observed_close.notna()
    required_history = int(np.ceil(253 * min_coverage))
    history_ok = observed.rolling(253, min_periods=253).sum() >= required_history
    recent_ok = (
        observed.rolling(recent_window, min_periods=recent_window).sum()
        >= min_recent_trading_days
    )
    return observed & history_ok & recent_ok

def calculate_cross_sectional_rating(
    raw_score: pd.DataFrame,
    valid_mask: pd.DataFrame | None = None,
) -> pd.DataFrame:
    """逐交易日计算真实横截面百分位评分，范围1~99，分数越高越强。"""
    scores = raw_score.where(valid_mask) if valid_mask is not None else raw_score
    pct_rank = scores.rank(axis=1, method="average", pct=True, na_option="keep")
    rating = np.ceil(pct_rank * 99.0).clip(1, 99)
    return rating.astype("Float64")

def calculate_thresholds(raw_score: pd.DataFrame, valid_mask: pd.DataFrame) -> pd.DataFrame:
    """阈值只用于审计与兼容旧DLL，不参与真实Rating计算。"""
    scores = raw_score.where(valid_mask)
    quantiles = {
        "q99": 0.99,
        "q90": 0.90,
        "q70": 0.70,
        "q50": 0.50,
        "q30": 0.30,
        "q10": 0.10,
        "q01": 0.01,
    }
    result = pd.DataFrame(index=scores.index)
    result["sample_count"] = scores.count(axis=1)
    for name, q in quantiles.items():
        result[name] = scores.quantile(q, axis=1)
    return result

def calculate_all(
    observed_close: pd.DataFrame,
    max_fill_days: int = 5,
    min_coverage: float = 0.95,
    recent_window: int = 20,
    min_recent_trading_days: int = 15,
) -> tuple[pd.DataFrame, pd.DataFrame, pd.DataFrame, pd.DataFrame]:
    observed_close = observed_close.sort_index().astype(float)
    calculation_close = observed_close.ffill(limit=max_fill_days)
    valid_mask = calculate_valid_mask(
        observed_close,
        min_coverage=min_coverage,
        recent_window=recent_window,
        min_recent_trading_days=min_recent_trading_days,
    )
    raw_score = calculate_raw_score(calculation_close)
    valid_mask &= raw_score.notna()
    rating = calculate_cross_sectional_rating(raw_score, valid_mask)
    thresholds = calculate_thresholds(raw_score, valid_mask)
    return raw_score, rating, valid_mask, thresholds
```

### rs_rating_tq.py｜TQ内部运行主程序

```python
from __future__ import annotations

import json
import sqlite3
import sys
import time
from pathlib import Path

import numpy as np
import pandas as pd

from rs_core import calculate_all

BASE_DIR = Path(__file__).resolve().parent

# ============================================================
# 内部运行参数：TQ内部运行不能接收命令行参数，直接在这里修改。
# 测试模式：LIMIT_STOCKS=30、HISTORY_BARS=400
# 正式模式：LIMIT_STOCKS=None、HISTORY_BARS=1500
# ============================================================
LIMIT_STOCKS: int | None = 30
HISTORY_BARS = 400
NO_PUSH = False

CONFIG = {
    "market": "5",
    "allowed_suffixes": [".SH", ".SZ"],
    "period": "1d",
    "dividend_type": "back",
    "history_bars": HISTORY_BARS,
    "batch_size": 300,
    "max_fill_days": 5,
    "min_coverage": 0.95,
    "recent_window": 20,
    "min_recent_trading_days": 15,
    "push_days": 500,
    "push_pause_seconds": 0.02,
    "output_dir": "output",
    "sqlite_file": "rs_rating.db",
    "latest_csv": "rs_rating_latest.csv",
    "threshold_csv": "rs_thresholds_history.csv",
}

def chunks(items: list[str], size: int):
    for i in range(0, len(items), size):
        yield items[i : i + size]

def normalize_close(payload, requested_codes: list[str]) -> pd.DataFrame:
    """兼容TQ常见的 {'Close': DataFrame} 返回结构。"""
    if not isinstance(payload, dict) or "Close" not in payload:
        raise ValueError("get_market_data返回结果中没有Close字段")
    close = payload["Close"]
    if not isinstance(close, pd.DataFrame):
        close = pd.DataFrame(close)

    requested = set(requested_codes)
    column_hits = len(requested.intersection(map(str, close.columns)))
    index_hits = len(requested.intersection(map(str, close.index)))
    if index_hits > column_hits:
        close = close.T

    close.columns = close.columns.map(str)
    close = close.loc[:, [c for c in close.columns if c in requested]]
    close.index = pd.to_datetime(close.index.astype(str), errors="coerce")
    close = close.loc[close.index.notna()]
    close = close[~close.index.duplicated(keep="last")]
    return close.apply(pd.to_numeric, errors="coerce").sort_index()

def acquire_close(tq, config: dict, limit_stocks: int | None = None) -> pd.DataFrame:
    all_codes = list(tq.get_stock_list(market=config["market"]))
    suffixes = tuple(config["allowed_suffixes"])
    codes = [str(c) for c in all_codes if str(c).endswith(suffixes)]
    if limit_stocks:
        codes = codes[:limit_stocks]
    if not codes:
        raise RuntimeError("没有取得符合市场后缀配置的股票代码")

    print(f"股票数量：{len(codes)}")
    frames: list[pd.DataFrame] = []
    batches = list(chunks(codes, int(config["batch_size"])))
    for batch_no, batch in enumerate(batches, 1):
        print(f"读取行情批次 {batch_no}/{len(batches)}，股票数={len(batch)}")
        payload = tq.get_market_data(
            field_list=["Close"],
            stock_list=batch,
            count=int(config["history_bars"]),
            dividend_type=config["dividend_type"],
            period=config["period"],
            fill_data=False,
        )
        frames.append(normalize_close(payload, batch))

    close = pd.concat(frames, axis=1).sort_index()
    close = close.loc[:, ~close.columns.duplicated(keep="last")]
    if close.empty:
        raise RuntimeError("行情结果为空，请先在通达信客户端下载盘后数据")
    print(f"行情范围：{close.index.min().date()} ~ {close.index.max().date()}")
    return close

def to_long_results(
    raw: pd.DataFrame,
    rating: pd.DataFrame,
    valid: pd.DataFrame,
) -> pd.DataFrame:
    records = []
    for code in rating.columns:
        frame = pd.DataFrame(
            {
                "trade_date": rating.index.strftime("%Y%m%d"),
                "stock_code": code,
                "rs_rating": rating[code].astype("Float64"),
                "raw_score": raw[code],
                "valid": valid[code].astype(int),
            }
        )
        frame = frame.loc[frame["rs_rating"].notna()]
        records.append(frame)
    if not records:
        return pd.DataFrame(columns=["trade_date", "stock_code", "rs_rating", "raw_score", "valid"])
    result = pd.concat(records, ignore_index=True)
    result["rs_rating"] = result["rs_rating"].astype(int)
    return result

def persist_results(
    config: dict,
    raw: pd.DataFrame,
    rating: pd.DataFrame,
    valid: pd.DataFrame,
    thresholds: pd.DataFrame,
) -> tuple[Path, Path, Path]:
    output_dir = BASE_DIR / config["output_dir"]
    output_dir.mkdir(parents=True, exist_ok=True)
    db_path = output_dir / config["sqlite_file"]
    latest_csv = output_dir / config["latest_csv"]
    threshold_csv = output_dir / config["threshold_csv"]

    long_results = to_long_results(raw, rating, valid)
    threshold_out = thresholds.copy()
    threshold_out.insert(0, "trade_date", threshold_out.index.strftime("%Y%m%d"))

    # Windows不允许os.replace()覆盖正在被数据库查看器占用的文件。
    # 改为直接更新固定数据库，并使用WAL和busy_timeout兼容并发读取。
    conn = sqlite3.connect(db_path, timeout=60)
    try:
        conn.execute("PRAGMA busy_timeout = 60000")
        conn.execute("PRAGMA journal_mode = WAL")
        conn.execute("PRAGMA synchronous = NORMAL")
        long_results.to_sql("rs_rating", conn, index=False, if_exists="replace")
        threshold_out.to_sql("rs_thresholds", conn, index=False, if_exists="replace")
        conn.execute("CREATE INDEX idx_rating_code_date ON rs_rating(stock_code, trade_date)")
        conn.execute("CREATE INDEX idx_rating_date ON rs_rating(trade_date)")
        conn.execute("CREATE UNIQUE INDEX idx_threshold_date ON rs_thresholds(trade_date)")
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

    latest_date = long_results["trade_date"].max()
    latest = long_results.loc[long_results["trade_date"] == latest_date].copy()
    latest = latest.sort_values(["rs_rating", "raw_score"], ascending=False)
    latest.to_csv(latest_csv, index=False, encoding="utf-8-sig")
    threshold_out.to_csv(threshold_csv, index=False, encoding="utf-8-sig")
    print(f"有效最新样本：{len(latest)}，日期={latest_date}")
    return db_path, latest_csv, threshold_csv

def push_to_tdx(tq, config: dict, raw: pd.DataFrame, rating: pd.DataFrame) -> dict:
    push_days = int(config["push_days"])
    pause = float(config["push_pause_seconds"])
    success = 0
    failed: dict[str, str] = {}
    first_response_printed = False

    for idx, code in enumerate(rating.columns, 1):
        frame = pd.DataFrame({"rating": rating[code], "raw": raw[code]}).dropna().tail(push_days)
        if frame.empty:
            continue
        # 日线SIGNALS_TQ按交易日键匹配。官方日线示例使用YYYYMMDD；
        # 若附加150000，部分客户端版本不会将其匹配到日K线。
        time_list = [d.strftime("%Y%m%d") for d in frame.index]
        # 与官方stratexldata.py保持一致：每行固定提供16个数字字符串。
        # 当前使用ID 1~3，其余ID预留为0。
        data_list = []
        for rating_value, raw_value in frame[["rating", "raw"]].itertuples(index=False, name=None):
            row = [str(int(rating_value)), f"{float(raw_value):.6f}", "1"]
            row.extend(["0"] * 13)
            data_list.append(row)
        try:
            result = tq.send_bt_data(
                stock_code=code,
                time_list=time_list,
                data_list=data_list,
                count=len(time_list),
            )
            if not first_response_printed:
                print(f"首只股票推送样例：code={code}, count={len(time_list)}")
                print(f"首条日期：{time_list[0]}，末条日期：{time_list[-1]}")
                print(f"末条数据：{data_list[-1]}")
                print(f"TQ原始返回：{result!r}")
                first_response_printed = True
            if isinstance(result, dict) and result.get("ErrorId") not in (None, 0, "0"):
                raise RuntimeError(str(result))
            success += 1
        except Exception as exc:
            failed[code] = str(exc)
        if idx % 100 == 0:
            print(f"推送进度：{idx}/{len(rating.columns)}，成功={success}，失败={len(failed)}")
        if pause > 0:
            time.sleep(pause)

    return {"success": success, "failed": failed}

def validate_distribution(rating: pd.DataFrame, thresholds: pd.DataFrame) -> None:
    latest = rating.iloc[-1].dropna().astype(float)
    if latest.empty:
        raise RuntimeError("最新交易日没有有效评分")
    median = float(latest.median())
    high90_ratio = float((latest >= 90).mean())
    high70_ratio = float((latest >= 70).mean())
    print(
        "评分校验："
        f"min={latest.min():.0f}, median={median:.1f}, max={latest.max():.0f}, "
        f">=90占比={high90_ratio:.2%}, >=70占比={high70_ratio:.2%}"
    )
    if not (45 <= median <= 55):
        raise RuntimeError("评分中位数异常，请检查样本和计算口径")
    if int(thresholds.iloc[-1]["sample_count"]) != len(latest):
        raise RuntimeError("阈值样本数与评分样本数不一致")

def main():
    config = dict(CONFIG)
    print(
        "内部运行参数："
        f"LIMIT_STOCKS={LIMIT_STOCKS}, "
        f"HISTORY_BARS={config['history_bars']}, "
        f"NO_PUSH={NO_PUSH}"
    )

    try:
        from tqcenter import tq
    except ImportError as exc:
        raise RuntimeError("未找到tqcenter；请在通达信TQ量化平台环境中运行") from exc

    tq.initialize(__file__)
    try:
        close = acquire_close(tq, config, LIMIT_STOCKS)
        raw, rating, valid, thresholds = calculate_all(
            close,
            max_fill_days=int(config["max_fill_days"]),
            min_coverage=float(config["min_coverage"]),
            recent_window=int(config["recent_window"]),
            min_recent_trading_days=int(config["min_recent_trading_days"]),
        )
        validate_distribution(rating, thresholds)
        paths = persist_results(config, raw, rating, valid, thresholds)
        print("保存完成：", *(str(p) for p in paths), sep="\n- ")

        if not NO_PUSH:
            push_result = push_to_tdx(tq, config, raw, rating)
            print(f"推送完成：成功={push_result['success']}，失败={len(push_result['failed'])}")
            if push_result["failed"]:
                error_path = BASE_DIR / config["output_dir"] / "push_errors.json"
                error_path.write_text(
                    json.dumps(push_result["failed"], ensure_ascii=False, indent=2),
                    encoding="utf-8",
                )
                print(f"失败明细：{error_path}")
    finally:
        close_func = getattr(tq, "close", None)
        if callable(close_func):
            close_func()

if __name__ == "__main__":
    try:
        main()
    except Exception as exc:
        print(f"ERROR: {exc}", file=sys.stderr)
        raise
```

### RSRATING_EOD.tdx｜通达信副图公式

```
{======================================================}
{ RS Rating EOD｜A股真实横截面评分 }
{ 数据来源：Python通过TQ send_bt_data推送 }
{ ID1=RS Rating；ID2=Raw Score；ID3=Valid }
{======================================================}

RSRATING:SIGNALS_TQ(1,0),COLORWHITE,LINETHICK2;
RAWSCORE:=SIGNALS_TQ(2,0);
VALID:=SIGNALS_TQ(3,0);

STRONG:85,COLORRED,LINETHICK1;
GOOD:70,COLORYELLOW,LINETHICK1;
MID:50,COLORGRAY,POINTDOT;
WEAK:30,COLORGREEN,LINETHICK1;

STICKLINE(VALID=1 AND RSRATING>=85,50,RSRATING,4,0),COLORRED;
STICKLINE(VALID=1 AND RSRATING>=70 AND RSRATING<85,50,RSRATING,4,0),COLORMAGENTA;
STICKLINE(VALID=1 AND RSRATING>=50 AND RSRATING<70,50,RSRATING,4,0),COLORYELLOW;
STICKLINE(VALID=1 AND RSRATING>=30 AND RSRATING<50,50,RSRATING,4,0),COLORBLUE;
STICKLINE(VALID=1 AND RSRATING<30,50,RSRATING,4,0),COLORGREEN;

DRAWTEXT(VALID=1 AND CROSS(RSRATING,85),88,'进入强势区'),COLORRED;
DRAWTEXT(VALID=1 AND CROSS(70,RSRATING),67,'跌出强势区'),COLORGREEN;

RATINGUP:=VALID=1 AND RSRATING>REF(RSRATING,1);
RATINGDOWN:=VALID=1 AND RSRATING<REF(RSRATING,1);

DRAWNUMBER(ISLASTBAR AND VALID=1,RSRATING,RSRATING),COLORWHITE;
```

### test_rs_core.py｜独立测试程序

```python
import numpy as np
import pandas as pd

from rs_core import calculate_all

def main():
    rng = np.random.default_rng(20260901)
    dates = pd.bdate_range("2022-01-03", periods=700)
    stocks = [f"{i:06d}.SZ" for i in range(1, 101)]
    shocks = rng.normal(0.0004, 0.018, size=(len(dates), len(stocks)))
    drifts = np.linspace(-0.0005, 0.0015, len(stocks))[None, :]
    close = pd.DataFrame(100 * np.exp(np.cumsum(shocks + drifts, axis=0)), index=dates, columns=stocks)

    raw, rating, valid, thresholds = calculate_all(close)
    latest = rating.iloc[-1].dropna()
    assert len(latest) == 100
    assert latest.min() >= 1 and latest.max() <= 99
    assert 49 <= float(latest.median()) <= 51
    assert thresholds.iloc[-1]["sample_count"] == 100
    strongest = raw.iloc[-1].idxmax()
    assert rating.iloc[-1][strongest] == 99
    print("PASS")
    print({
        "sample_count": int(len(latest)),
        "min": int(latest.min()),
        "median": float(latest.median()),
        "max": int(latest.max()),
        "strongest": strongest,
    })

if __name__ == "__main__":
    main()
```

## 2. 目标与技术决策

目标：由Python对当日合格A股完成横截面排名，向通达信发送每只股票的历史RS Rating，而非用固定美股阈值近似评分。

```
通达信TQ内部运行Python
→ 获取后复权日线
→ 逐日计算每只股票Raw Score
→ 当日有效样本内排名
→ SQLite与CSV存档
→ tq.send_bt_data()按股票推送
→ SIGNALS_TQ()在日线副图显示
```

- 采用方案B：直接计算真实样本内评分，不依赖七档阈值插值。
- 旧DLL只作历史对照，不在正式评分显示链路中。
- CSV是审计输出，不是Python与公式之间的主通信通道。
- 七档阈值仍导出供检查，但不参与Rating计算。
- “真实评分”指所选有效样本的精确百分位，不表示与IBD专有评分完全一致。

背景说明：[RS Rating 指标详解](https://app.notion.com/p/RS-Rating-c9b1c1df7ffa49c686b492149ae8f363?pvs=21)

## 3. 已验证环境与运行入口

本次用户环境：

```
通达信目录：D:\new_tdx64
项目目录：D:\new_tdx64\PYPlugins\user\tdx_project\rs_rating_tq
Python依赖：numpy、pandas、通达信提供的tqcenter
```

本机Python精确版本、通达信版本号和依赖版本未记录，不能声称锁定了完整可复现环境。

关键结论：在用户此次环境中，必须从TQ内部运行才能让界面读取信号。外部Python运行时，接口曾返回“发送TQ数据成功”，但日线副图仍为0。该结论是本机实测结果，不推广为所有版本的绝对限制。

内部运行初始化：

```python
tq.initialize(__file__)
```

内部运行不接受原有外部命令行参数，因此最终版移除了argparse，并把运行参数写到程序顶部。无需config.json，也无需传入--limit-stocks等参数。两个Python模块需放在同一可导入目录。

## 4. 配置与启动

### 默认小样本配置（归档代码原样保留）

```python
LIMIT_STOCKS: int | None = 30
HISTORY_BARS = 400
NO_PUSH = False
```

- LIMIT_STOCKS：取股票列表前N只；None表示不限制。
- HISTORY_BARS：每只股票请求的历史日线数量。
- NO_PUSH：True只计算保存；False计算保存后推送。

小样本得到的是30只股票内部的排名，不是全市场排名，不能直接用于正式选股。

### 全市场配置（建议操作，未记录最终验收）

在工作副本修改：

```python
LIMIT_STOCKS: int | None = None
HISTORY_BARS = 1500
NO_PUSH = False
```

其他内置默认项：

```python
"market": "5"
"allowed_suffixes": [".SH", ".SZ"]
"period": "1d"
"dividend_type": "back"
"batch_size": 300
"max_fill_days": 5
"min_coverage": 0.95
"recent_window": 20
"min_recent_trading_days": 15
"push_days": 500
"push_pause_seconds": 0.02
```

操作流程：

1. 启动并登录通达信，下载完整盘后日线。
2. 解压稳定包，保持rs_rating_[tq.py](http://tq.py)与rs_[core.py](http://core.py)同目录。
3. 从已验证的TQ内部入口运行rs_rating_[tq.py](http://tq.py)，不传命令行参数。
4. 在公式管理器新建日线副图指标，粘贴RSRATING_EOD.tdx。
5. 打开实际已推送的股票日线，将最新评分与CSV对照。
6. 先保留测试日志，再在工作副本扩容；不要覆盖归档ZIP。

建议仅在盘后数据完整时运行。程序尚未实现盘中自动剔除未收盘日K线。

## 5. 算法及样本口径

```
Raw Score = 100 × (
  0.40 × P(t) / P(t-63)
+ 0.20 × P(t) / P(t-126)
+ 0.20 × P(t) / P(t-189)
+ 0.20 × P(t) / P(t-252)
)

pct_rank = 当日有效股票Raw Score的升序平均排名 / 当日有效数量
RS Rating = ceil(pct_rank × 99)，限制为1～99
```

实现使用pandas横截面rank，method="average"处理并列。各周期使用同一后复权价格口径。横截面排名无需再除以共同基准；同日所有股票除以同一个正数不会改变顺序。

现有代码规则：

- 数据来源为get_stock_list(market="5")，[随后按.SH/.SZ后缀筛选。](http://随后按.SH/.SZ后缀筛选。)
- 不主动剔除ST；未完成独立证券主数据核验。
- 按合并后日期索引计算回看周期。
- 253行窗口内非空价格覆盖率至少95%。
- 最近20行非空价格至少15行。
- 当日原始收盘价必须非空。
- 最多向前填充5行用于价格比计算。
- 价格比需为正且有限；Raw Score有效后才能排名。

七档审计分位为99%、90%、70%、50%、30%、10%、1%。

## 6. TQ序列数据约定

```python
time_list = [d.strftime("%Y%m%d") for d in frame.index]

row = [
    str(int(rating_value)),
    f"{float(raw_value):.6f}",
    "1",
]
row.extend(["0"] * 13)

tq.send_bt_data(
    stock_code=code,
    time_list=time_list,
    data_list=data_list,
    count=len(time_list),
)
```

- 每行固定16列数字字符串。
- ID1：RS Rating。
- ID2：Raw Score（本版为加权价格比×100，不是相对沪深300比值）。
- ID3：Valid；当前只推送有效记录，固定为1。
- ID4～16：填0预留。
- 日线日期使用YYYYMMDD，count与行数一致。
- 默认每股推送最近500条有效记录，而非全部计算历史。

公式关键入口：

```
RSRATING:SIGNALS_TQ(1,0),COLORWHITE,LINETHICK2;
RAWSCORE:=SIGNALS_TQ(2,0);
VALID:=SIGNALS_TQ(3,0);
```

完整显示公式以附件中的RSRATING_EOD.tdx为准。TYPE=0不平滑；诊断可使用TYPE=2让缺失返回0，但不能把0解释成有效Rating。

## 7. 数据存储与重运行行为

输出到脚本目录下output：

- rs_rating.db：rs_rating评分表、rs_thresholds阈值表及索引。
- rs_rating_latest.csv：本次计算所得最新有效交易日的评分。
- rs_thresholds_history.csv：本次计算窗口的历史阈值与样本数。
- push_errors.json：本次存在异常时写出推送失败信息。

重要：本版每次替换数据库表和CSV内容，并不是增量追加长期历史。调整样本数或历史长度后，旧结果可能被本次窗口覆盖；重跑前应备份output目录。先前遗留的push_errors.json不会在本次全部成功后自动删除，需结合时间和运行日志判断。

SQLite采用固定数据库路径、WAL及60秒锁等待，并在finally显式关闭连接。数据库查看器持有写锁时仍可能失败。CSV在Excel中被占用也可能妨碍覆盖。

## 8. 开发排错记录与结论

### V2：固定阈值DLL

移植TradingView固定阈值作近似评分，不是真实A股横截面排名。保留参考，不作为当前正式输出。

### V3：Python排名及TQ推送

实现行情批量读取、横截面评分、SQLite/CSV和TQ公式显示链路。

### V3.1：Windows数据库文件替换报错

旧程序使用SQLite连接上下文后执行os.replace，出现WinError 32。sqlite3连接上下文负责事务，不保证退出时关闭连接，自身仍持有文件也可能是原因，不能仅归咎于外部查看器。修订为固定数据库写入、显式关闭及WAL；用户随后确认保存成功、30只股票推送无异常。

### V3.2 / V3.3：日期与列格式对齐

采用官方日线示例的8位日期，并补足16列，同时打印原始响应。这些修改在最终版保留，但单独修改后未解决界面为0，不能当作已证实根因。

### V3.4：外部初始化文件名实验

外部运行改为只传文件名，仍未解决显示。不是最终稳定版，不应覆盖V3.3 Internal。

### V3.3 Internal：实际解决方案

用户验证TQ内部运行可显示。把外部参数写入脚本、恢复initialize(**file**)，2026-09-02确认功能及界面正常。

早先“run_id需手工绑定”“必须选中某策略记录”等判断未得到证实，不列为操作要求。send_warn预警可用仅能证明预警通道正常，不等于序列公式可读。SIGNALS_USER及CSV+DLL桥接均未成为最终方案。

## 9. 验收状态

- [x]  Python语法检查及原有合成数据测试通过。
- [x]  30只股票行情、计算、SQLite/CSV保存成功。
- [x]  TQ接口返回发送成功。
- [x]  用户确认内部运行后副图显示正常。
- [ ]  全市场长期稳定性、性能与内存验收。
- [ ]  全市场随机股票逐日CSV/界面一致性审计。
- [ ]  历史时点股票池及退市样本补全。
- [ ]  自动盘后调度、重试及增量存储。

参考联调日志日期为2026-09-01，样本30只、400根日线；min=4、median=51.5、max=99。小样本最弱评分为ceil(99/30)=4是当前分桶定义的结果，不是异常。并列较多时不应强制要求均匀分布。

## 10. 已知限制与后续工作（未修改稳定代码）

1. 当前股票名单回溯历史存在幸存者偏差；逐日rank不自动消除该偏差。需历史时点成员及退市数据后才能宣称无偏回测。
2. 日期轴来自所取股票行情的并集，未独立调用官方交易日历。不能将其描述为已完成严格日历校验。
3. “非空价格”不等于真实成交日。若TQ停牌日返回陈旧非空价格，覆盖率规则无法独立识别停牌；需交易状态/成交量进一步验证。
4. 253日95%覆盖率不是精确上市天数校验；可能允许窗口前部存在缺失，需主数据上市日期做严格筛选。
5. 当前有效记录缺失时不推送Valid=0，也未验证旧TQ记录的覆盖/清除语义；需测试停牌、失效及重跑后的旧值问题。
6. 推送“成功”统计主要检查返回字典中的ErrorId；无明确错误码的异常返回可能被计为成功，需后续强化。
7. 全量重算覆盖历史表，未实现跨两张表与CSV的整体原子发布。推送与数据库保存不是同一事务。
8. 默认只推送500条有效记录；1500根行情不会全部显示为1500天有效评分。
9. 评分是动量排名，不是收益保证；70/85/90等线是筛选参考，不是独立买卖指令。

## 11. 版本保护与回滚

- 本页附件是冻结基线，不直接改写原包。
- 后续功能在新目录、新版本开发，记录配置、代码差异及验收结果。
- 变更前备份代码、output数据和通达信公式。
- 失败时恢复附件中的同名文件，在TQ内部运行默认30只/400根配置验证；不要恢复外部参数入口。
- 每个新版本需同时核验Python结果、TQ返回和通达信实际显示，不能只以“发送成功”作为完成标准。

## 12. 官方参考

- [TQ策略与stratexldata.py示例](https://help.tdx.com.cn/quant/docs/markdown/gzh0122inweixinwenz/gzh20260122wzlz.html#stratexldata-py)
- [公式与Python双向互通](https://help.tdx.com.cn/quant/docs/markdown/gzh0122inweixinwenz/gzh20260302wzlz.html)