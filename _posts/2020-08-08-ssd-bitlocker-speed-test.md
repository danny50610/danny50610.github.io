---
layout: post
title:  "紀錄 SSD 有無啟用 Bitlocker 的速度差異"
date:   2020-08-08 20:48:00 +0800
tags:   [SSD, Bitlocker]
---

筆電原本的 SSD 只有 120 G，前陣子開始覺得空間不太夠用，想要換大一點的 SSD。
經過研究和採買後，想說可以做速度測試，除了比較新舊差異外，還有看看 Bitlocker 的影響程度。

<!--more-->

舊 SSD 規格：SK hynix SC308 128GB M.2 2280 SATA (MLC)

新 SSD 規格：Micron Crucial MX500 500GB M.2 2280 SATA (TLC)

測試軟體：CrystalDiskMark 7.0.0

測試的時候，舊 SSD 已經使用 93% 的儲存空間了，以及兩顆 SSD 是在不同的 Windows 10 上測試的，所以不適合直接比較。

底下是測試結果，就不寫結論了。

{% include image.html url="/assets/img/2020-08-08/old-ssd-without-bitlocker.jpg" description="舊 SSD 不啟用 Bitlocker" %}

{% include image.html url="/assets/img/2020-08-08/old-ssd-with-bitlocker.jpg" description="舊 SSD 啟用 Bitlocker" %}

{% include image.html url="/assets/img/2020-08-08/new-ssd-without-bitlocker.jpg" description="新 SSD 不啟用 Bitlocker" %}

{% include image.html url="/assets/img/2020-08-08/new-ssd-with-bitlocker.jpg" description="新 SSD 啟用 Bitlocker" %}