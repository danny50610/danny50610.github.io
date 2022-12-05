---
layout: post
title:  "Android 12 分離 Wi-Fi 和 行動網路 的快速按鈕筆記"
date:   2022-12-04 21:52:00 +0800
tags:   [android 12, android]
---

Android 12 把「Wi-Fi」和「行動網路」的快速按鈕合併在一起了  
變成一個「網際網路」的按鈕

{% include image.html url="/assets/img/2022-12-05/before.png" class="w-50" %}

變成開關行動網路要多按一次  
完全無法理解為什麼要這樣做...

有鑑於網路上的教學都缺一些步驟，決定自己記錄一下

<!--more-->

接下來的指令是在 **Windows** 使用

1. 首先先截圖備份按鈕的位置，免得不小心重設成原本的設定

2. 首先開啟 USB 偵錯，並把把手機接上電腦

3. 開啟 Terminal，並移動到 adb 所在的位置
```
cd "%LocalAppData%\Android\Sdk\platform-tools"
```

4. 接著執行這兩行
```
adb shell settings put global settings_provider_model false
adb shell settings put secure sysui_qs_tiles wifi,cell,$(settings get secure sysui_qs_tiles)
```

5. 大功告成，在手動移除「網際網路」的按鈕即可

{% include image.html url="/assets/img/2022-12-05/after.png" class="w-50" %}
