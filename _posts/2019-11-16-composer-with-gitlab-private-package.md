---
layout: post
title:  "讓 composer 使用位在 Gitlab 的 private package"
date:   2019-11-16 19:41:00 +0800
---

之前發現兩個自己維護的網站有重複的 code  
想說提出成 package 來重複利用

不過由於都不是 open source 的網站，所以 package 先弄成 private  
這導致 composer 設定上比較麻煩  
雖然最後看起來很簡單，但這是經過不少錯誤的嘗試才成功  
這邊寫個文章記錄下來

<!--more-->

---

## 目標
假設有一個放在 Gitlab 的 private repository  
名稱是：`danny50610/website-core`  
最少設有一個 tag (假設是：`0.1`)  

希望使用 `Deploy Keys` 存取

目標是想讓 `danny50610/mywebsite` 使用 `danny50610/website-core`  
讓開發人員用的電腦、CI 和 Server 都能使用

## Deploy Keys
首先，由於 `danny50610/website-core` 是 private repository  
要先新增 `Deploy Keys`，每一台要使用的電腦都要建立，並新增 public key 到 Gitlab 上

這邊不列出指令，可以參考 Gitlab 的<a href="https://docs.gitlab.com/ee/ssh/#generating-a-new-ssh-key-pair" target="_blank" rel="noopener">教學</a>

## composer require
在 `danny50610/mywebsite` 的 `composer.json` 加上
```
{
    ...
    "repositories": [
        {
            "type": "git",
            "url":  "git@gitlab.com:danny50610/website-core.git"
        }
    ],
    ...
}
```
`type` 請使用 `git`，不要用 `vcs`、`gitlab`，這樣 `composer` 才會使用 ssh  
`url` 是 repository 的 clone url (ssh)

接著執行，就大功告成：
```
composer require danny50610/website-core
```

## 備註
如果之前有錯誤的嘗試，先移除先前的設定，並把 `composer.json` 和 `composer.lock` 回復成原本的樣子

因為自己遇到 `composer install` 突然要求輸入 Gitlab 的 password  
後來發現是 `composer.lock` 的 `danny50610/website-core` 不小心讓他存到 https 的下載地址  
先 `composer remove danny50610/website-core` 再 `composer require danny50610/website-core` 就沒問題了
