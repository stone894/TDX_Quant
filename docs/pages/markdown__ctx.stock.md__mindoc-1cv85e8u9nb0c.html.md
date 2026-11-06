---
title: "初始化initialize"
source_url: https://help.tdx.com.cn/quant/docs/markdown/ctx.stock.md/mindoc-1cv85e8u9nb0c.html
fetched_at: 2026-11-06T06:02:44Z
---

initialize(__file__) #所有策略连接通达信客户端都必须调用此函数进行初始化

from tqcenter import tq tq.initialize(__file__)

1.tqcenter本地客户端模式下，使用tq前需要调用这个函数进行初始化。 2.同一个py文件，不能initialize多次。同一个位置的有initialize的py文件也不能同时运行，第二次执行initialize会提示错误。

← 快速开始第一个策略 订阅行情subscribe_hq →
