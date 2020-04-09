---
layout: post
title:  "VirtualBox 與 USBPcap 衝突，導致 USB 裝置無法掛載到虛擬機器上"
date:   2020-04-09 20:51:00 +0800
tags:   [VirtualBox, USBPcap]
---

這篇只是記錄一下  
如果遇到 VirtualBox 無法使用 USB 裝置，有可能是因為 USBPcap 的關係  
通常 USBPcap 可能會跟 Wireshark 一起安裝

<!--more-->

如果把 USBPcap 移除的話，就可以正常使用 VirtualBox 的 USB  
(記得重新開機，或是關機再開機，包含 Host 和 Guest)

可能的錯誤訊息：
```
USB device 'XXXXX' with UUID {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx} is busy with a previous request. Please try again later.
```

參考資料：  
<a href="https://www.zachpfeffer.com/single-post/A-fix-for-a-USB-drive-or-any-USB-device-not-working-in-VirtualBox" target="_blank" rel="noopener">https://www.zachpfeffer.com/single-post/A-fix-for-a-USB-drive-or-any-USB-device-not-working-in-VirtualBox</a>  
<a href="https://github.com/desowin/usbpcap/issues/62" target="_blank" rel="noopener">https://github.com/desowin/usbpcap/issues/62</a>

