---
layout: post
title:  "Android 國家級警報不會響可能解決方式"
date:   2022-07-31 14:53:00 +0800
tags:   [android, Emergency Alerts]
---

7/25 剛好是北部的萬安演習，結果有收到國家級警報，但手機卻沒有發出警示音  
沒有警示音就失去警報的意義了。研究一下發現可能是因為開啟「零打擾」的關係  
下面記錄一下解決方式

{% include image.html url="/assets/img/2022-07-31/國家級警報.jpg" class="w-50" %}

<!--more-->

備註：手機作業系統是使用 Android 12  

首先先進到「設定」  
{% include image.html url="/assets/img/2022-07-31/設定.jpg" class="w-50" %}

點選「通知」
{% include image.html url="/assets/img/2022-07-31/通知.jpg" class="w-50" %}

往下拉，點選「零打擾」
{% include image.html url="/assets/img/2022-07-31/零打擾.jpg" class="w-50" %}

點選例外項目的「應用程式」
{% include image.html url="/assets/img/2022-07-31/零打擾-應用程式.jpg" class="w-50" %}

看裡面是否有「無限緊急警報」
{% include image.html url="/assets/img/2022-07-31/零打擾-應用程式-例外.jpg" class="w-50" %}

沒有的話點下面的「新增應用程式」，把「無線緊急警報」加進去

這樣應該就會有警示音了

## 2025-06-18 補充

警示提醒這邊的設定也可以檢查一下

{% include image.html url="/assets/img/2022-07-31/警示提醒.jpg" class="w-50" %}