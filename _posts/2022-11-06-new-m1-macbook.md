---
layout: post
title:  "新 Macbook 的 Laravel 效能簡易測試"
date:   2022-11-06 23:43:00 +0800
tags:   [macbook, m1, docker, Vagrant, VirtualBox]
---

最近決定狠下心買新的 Macbook 了  
雖然目前原本這台 Macbook 算是合乎使用，但有時真的會慢到受不了  
而且也會有時還會很燙

不過都花錢買新 Macbook 了，總要測試一下到底快了多少，確認一下換電腦的決定是好的  
(雖然接下來比較沒有一個基準點就是，體感比較重要)

<!--more-->

首先先列出新舊 Macbook 的規格：

### 舊 Macbook (Macbook Air 2018) (Intel)
{% include image.html url="/assets/img/2022-11-06/macbook-air.png" class="w-50" %}

### 新 Macbook (Macbook Pro 2021) (M1 Pro)
{% include image.html url="/assets/img/2022-11-06/macbook-m1-pro.png" class="w-50" %}
CPU 是 8 核心 (6 + 2)


## 使用比較
在日常使用上，不管是開瀏覽器，或是網頁瀏覽，新 Macbook 都順上不少  
不過對於 Laravel 開發者來說，本地環境的 API 執行速度才是最重要的  
這也是接下來的測試目標

### 舊 Macbook (Macbook Air 2018)
在舊 Macbook 我是用 VirtualBox 跑 Web 環境  
CPU 固定在 2 顆，記憶體是 2048 MB，有開 Swap

PHP 版本是：
```bash
$ php -v
PHP 7.4.32 (cli) (built: Sep 29 2022 22:24:52) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.32, Copyright (c), by Zend Technologies
```

因為 VirtualBox Shared Folder 實在是很慢，所以專案資料夾有用 NFS 掛進虛擬機

測試方法是拿手邊的專案，重新整理幾次，最後平均 API 所花的時間

{% include image.html url="/assets/img/2022-11-06/macbook-air-http.png" %}

平均大概花費： 786 ms

### 新 Macbook (Macbook Pro 2021)
因為新 Macbook 變成 M1 晶片後，不能再使用 VirtualBox 跑  
所以改成 Docker 跑 Web 環境  
為了降低誤差(?)，Docker 的 VM 的 CPU 也固定在 2 顆，記憶體是 2048 MB，有開 Swap

PHP 版本是：
```bash
$ php -v
PHP 7.4.32 (cli) (built: Oct 28 2022 18:18:25) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.32, Copyright (c), by Zend Technologies
```

專案資料夾使用 Docker 的 Mount

測試方法一樣是拿手邊同一個專案，重新整理幾次，最後平均 API 所花的時間

{% include image.html url="/assets/img/2022-11-06/macbook-m1-pro-http.png" %}

平均大概花費： 195 ms

#### virtiofs

最近剛好發現 Docker Desktop for Mac 有一個實驗性功能  
啟用「virtiofs」可以加速 file sharing 的效能

{% include image.html url="/assets/img/2022-11-06/docker-virtiofs.png" %}

(參考資料：[https://www.docker.com/blog/speed-boost-achievement-unlocked-on-docker-desktop-4-6-for-mac/](https://www.docker.com/blog/speed-boost-achievement-unlocked-on-docker-desktop-4-6-for-mac/))

啟用後一樣是拿手邊同一個專案，重新整理幾次，最後平均 API 所花的時間

{% include image.html url="/assets/img/2022-11-06/macbook-m1-pro-virtiofs-http.png" %}

平均大概花費： 57 ms

## 結論

| 環境                                 | API 平均回應時間 |
|--------------------------------------|-------:|
| Macbook Air 2018 (VirtualBox)        | 786 ms |
| Macbook Pro 2021 (Docker)            | 195 ms |
| Macbook Pro 2021 (Docker + virtiofs) |  57 ms |

可以看到，速度是真的快上不少  
特別是用上 virtiofs 之後，平均可以壓到 100 ms 以下  
不過 virtiofs 是剛推出的功能，希望不會有太多 Bug
