---
layout: post
title:  "SSD SN750 突然離線，檔案系統進入 Read-only"
date:   2022-07-14 22:05:00 +0800
tags:   [ssd, SN750, westerndigital]
---

最近被我拿來當作 Server 的電腦，突然之間連不上  
本來以為是停電或網路斷線造成的  
沒想到實際一看才發現，Server 實際還開著，但噴了一堆 「can't open file - Read-only file system」 log  
也沒辦法正常關機

<!--more-->

本來以為要對 OS 的那顆 SSD 做 Filesystem 檢查，但沒想到重開機後就正常了 !?  
最後就放著再觀察看看

# 隔天
沒想才過一天多，上面的服務就突然連不上了  
緊急用 SSH 連進去 server 後，發現一樣是出現 「can't open file - Read-only file system」 log  
但趁這次還能連線，就趕快往前收集更多 Log，以下就是收集的成果(只留下最一開始出事的部分，應該也是最關鍵的)
```
Jul 03 22:04:18 celestia kernel: nvme nvme0: I/O 380 QID 9 timeout, aborting
Jul 03 22:04:18 celestia kernel: nvme nvme0: I/O 13 QID 3 timeout, aborting
Jul 03 22:04:20 celestia kernel: nvme nvme0: I/O 924 QID 1 timeout, aborting
Jul 03 22:04:20 celestia kernel: nvme nvme0: I/O 381 QID 9 timeout, aborting
Jul 03 22:04:20 celestia kernel: nvme nvme0: I/O 764 QID 11 timeout, aborting
Jul 03 22:04:49 celestia kernel: nvme nvme0: I/O 13 QID 3 timeout, reset controller
Jul 03 22:04:51 celestia kernel: nvme nvme0: I/O 0 QID 0 timeout, reset controller
Jul 03 22:06:01 celestia kernel: nvme nvme0: Device not ready; aborting reset, CSTS=0x1
Jul 03 22:06:23 celestia kernel: nvme nvme0: Abort status: 0x371
Jul 03 22:06:23 celestia kernel: nvme nvme0: Abort status: 0x371
Jul 03 22:06:23 celestia kernel: nvme nvme0: Abort status: 0x371
Jul 03 22:06:23 celestia kernel: nvme nvme0: Abort status: 0x371
Jul 03 22:06:23 celestia kernel: nvme nvme0: Abort status: 0x371
Jul 03 22:06:23 celestia kernel: nvme nvme0: Device not ready; aborting reset, CSTS=0x3
Jul 03 22:06:23 celestia kernel: nvme nvme0: Removing after probe failure status: -19
Jul 03 22:06:23 celestia kernel: Write-error on swap-device (253:0:4459408)
Jul 03 22:06:23 celestia kernel: Write-error on swap-device (253:0:4328680)
Jul 03 22:06:23 celestia kernel: EXT4-fs warning (device dm-1): ext4_end_bio:342: I/O error 10 writing to inode 4721403 starting block 3704633)
Jul 03 22:06:23 celestia kernel: EXT4-fs warning (device dm-13): ext4_end_bio:342: I/O error 10 writing to inode 2230028 starting block 12094863)
Jul 03 22:06:23 celestia kernel: Buffer I/O error on device dm-13, logical block 12094863
...
```

看到這 Log 心裡涼了一半，這顆 SSD (WD Black SN750 500G)才買不到半年就出問題  
而且更奇怪的是這麼會是現在才出問題

# Linux Kernel Bug ?
去拿去送修前決定先排除看看其他原因

第一個懷疑的就是會不會是 Linux Kernel 的 Bug，畢竟這幾個月有升級幾次 Kernel  
(但應該也不會這麼容易遇到 Bug 才對?)  
經過不斷 Google 後是有找到幾篇：  
<a href="https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1910866" target="_blank" rel="noopener">https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1910866</a>  
<a href="https://bugzilla.kernel.org/show_bug.cgi?id=195039" target="_blank" rel="noopener">https://bugzilla.kernel.org/show_bug.cgi?id=195039</a>

這些 issue 都有遇到類似的錯誤訊息  
但考量目前使用的 Kernel 版本是：`5.15.35-2-pve (build@proxmox)`  
距離 Issue 中遇到的版本都有一些差距  
而且這些 Issue 後來多少也有修正，所以直覺上覺得可能不是 Linux Kernel 的 Bug

# Firmware
那下一個懷疑的目標就是 Firmware  
經過不斷 Google 後發現這篇德文的文章：  
<a href="https://www.thomas-krenn.com/de/wiki/Western_Digital_SN640_Firmware_Updates_R1110021_und_R1410004" target="_blank" rel="noopener">https://www.thomas-krenn.com/de/wiki/Western_Digital_SN640_Firmware_Updates_R1110021_und_R1410004</a>

不過他是 SN640，但錯誤訊息蠻類似的  

但這邊遇到最大的問題，Western Digital 居然沒有 Linux 版的工具可以檢查或更新 Firmware  
也找不到 binary 檔手動更新...
本來想說有沒有做成開機 USB 的工具，但好像也找不到  
所以只能先暫緩這條路，畢竟不太好臨時找一台 Windows 電腦 + 有多的 M2 插槽的電腦  

# nvme_core.default_ps_max_latency_us
後來重新 Google 再加上仔細重讀 issue  
發現可能的原因或許是 SSD 的 APST 功能導致的  

再加上 Arch Linux 的 Wiki 也有在 Troubleshooting 提到這件事：  
<a href="https://wiki.archlinux.org/title/Solid_state_drive/NVMe#Troubleshooting" target="_blank" rel="noopener">https://wiki.archlinux.org/title/Solid_state_drive/NVMe</a>

錯誤訊息也蠻類似的，但好像又不太一樣  
但這是目前看起來比較容易嘗試的解法了

這邊參考 Wiki 的說法，另外個人覺得先禁用 P4 狀態，先設定 10000 看看 (P3 的 Ex_Lat 值)

```shell
root@celestia:~# smartctl -a /dev/nvme0
smartctl 7.2 2020-12-30 r5155 [x86_64-linux-5.15.39-1-pve] (local build)
Copyright (C) 2002-20, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Number:                       WDS500G3X0C-00SJG0
Firmware Version:                   111130WD
...

Supported Power States
St Op     Max   Active     Idle   RL RT WL WT  Ent_Lat  Ex_Lat
 0 +     5.50W       -        -    0  0  0  0        0       0
 1 +     3.50W       -        -    1  1  1  1        0       0
 2 +     3.00W       -        -    2  2  2  2        0       0
 3 -   0.0700W       -        -    3  3  3  3     4000   10000
 4 -   0.0035W       -        -    4  4  4  4     4000   40000
```

修改 `/etc/default/grub`
```
...
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet nvme_core.default_ps_max_latency_us=10000"
GRUB_CMDLINE_LINUX=""
...
```

接著執行 `sudo update-grub`，然後重開機試試

可以用 `cat /sys/module/nvme_core/parameters/default_ps_max_latency_us` 檢查設定

# 目前觀察 & 結論
從設定到這篇文章撰寫已經過了 5 天了，目前 Server 還在線上，希望這樣就解決了  
(所以到最後也不曉得有沒有新的 Firmware 可以用)

沒想到 2022 年了，SSD (NVMe 也算常見的了吧)，還會有需要特別為硬體調整參數的麻煩事  
而且居然有廠商不支援 Linux 工具，看來以後買 SSD 要多加確認了
