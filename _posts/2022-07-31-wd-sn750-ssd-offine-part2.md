---
layout: post
title:  "WD SSD SN750 突然離線，檔案系統進入 Read-only Part 2"
date:   2022-07-31 13:53:00 +0800
tags:   [ssd, SN750, westerndigital]
---

沒想到 7/30 又突然沒有回應，但這次沒看到 Log，螢幕也突然沒有輸出  
但還是先假設一樣 SSD 的問題

<!--more-->

這次決定將 `nvme_core.default_ps_max_latency_us` 設定成 `0` 了  
讓 APST 變成 Disable


可以用 nvme 確認
```
root@celestia:~# nvme get-feature /dev/nvme0 -f 0x0c -H
get-feature:0xc (Autonomous Power State Transition), Current value:00000000
        Autonomous Power State Transition Enable (APSTE): Disabled
        Auto PST Entries        .................
```

現在只能先觀察看看了...
