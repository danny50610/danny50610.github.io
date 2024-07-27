---
layout: post
title:  "快速替換成新的 Laravel 8 route 寫法的方法"
date:   2024-07-27 15:26:00 +0800
tags:   [laravel]
---

紀錄一下快速將 laravel route 中的

```
'PasswordController@getChangePassword'
```

換成

```
[PasswordController::class, 'getChangePassword']
```

的方法

<!--more-->

基本上就是使用 regex

搜尋: `'(\w+)@(\w+)'`  
取代: `\[$1::class, '$2'\]`

如果 Class 前面有資料夾的話，再手動取代掉就好  
然後使用 IDE 的功能 Import 這些 Controller
