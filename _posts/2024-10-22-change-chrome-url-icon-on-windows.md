---
layout: post
title:  "改變 Windows 上 Chrome 的 URL 捷徑圖示"
date:   2024-10-22 00:19:00 +0800
tags:   [chrome, windows]
---

最近 Chrome 升級到 130 後，在 Windows 上的儲存的 URL 捷徑居然變了

{% include image.html url="/assets/img/2024-10-22/old-new-icon.png" class="w-50" %}

<!--more-->

由於新 Icon 的變得非常小，感覺很不直觀

經過不斷的 Google 和 ChatGPT 後，終於到確認比較維護的改法

修改下列機碼值
```
Computer\HKEY_CLASSES_ROOT\ChromeHTML\DefaultIcon
```

從
```
C:\Program Files\Google\Chrome\Application\chrome.exe,10
```

改成
```
C:\Program Files\Google\Chrome\Application\chrome.exe,0
```

## 參考資料
* <a href="https://helpcenter.netwrix.com/bundle/PolicyPak/page/Content/PolicyPak/BrowserRouter/ShortcutIcons.htm" target="_blank" rel="noopener">https://helpcenter.netwrix.com/bundle/PolicyPak/page/Content/PolicyPak/BrowserRouter/ShortcutIcons.htm</a>
