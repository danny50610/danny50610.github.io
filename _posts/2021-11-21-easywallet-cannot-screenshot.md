---
layout: post
title:  "解決悠遊付無法截圖的奇怪 Bug"
date:   2021-11-21 17:52:00 +0800
tags:   [easywallet, android, screenshot]
---

環境：
 - 手機：Sony Xperia 5 II
 - OS：Android 11 (58.1.A.5.441)
 - 悠遊付：3.0.45_release_413

最近悠遊付截圖時會出現這個訊息：

<!--more-->

{% include image.html url="/assets/img/2021-11-21/cannot-screenshot.jpg" description="無法儲存螢幕擷取畫面" %}

看到**「這個應用程式或貴機構不允許擷取螢幕畫面」**這句話，直覺想到是悠遊付主動擋掉的。  
但在悠遊付裡面的找不到可以開啟的選項  

---

**後來突發奇想，會不會是權限不夠**  
於是開啟 APP 權限設定，先給予「儲存空間」的權限試試

{% include image.html url="/assets/img/2021-11-21/app-permission.jpg" description="給予「儲存空間」的權限" %}

然後嘗試截圖就成功了

{% include image.html url="/assets/img/2021-11-21/successful-screenshot.jpg" description="截圖成功" %}

如果遇到一樣的問題，可以試試這個解法  
不確定為什麼會跟「儲存空間」的權限有關就是，其他 APP 沒給權限也能截圖
