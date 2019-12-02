---
layout: post
title:  "IKEv2 VPN Server 筆記"
date:   2019-11-27 21:06:00 +0800
tags:   [strongswan, vpn, ikev2]
---

最近突然想要自架 VPN Server，於是參考這篇教學：  
<a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ikev2-vpn-server-with-strongswan-on-ubuntu-18-04-2" target="_blank" rel="noopener">https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ikev2-vpn-server-with-strongswan-on-ubuntu-18-04-2</a>

不過遇到預期之外的狀況，這邊記錄一下解決方法  
(這邊都是 Windows 10 的部分)

<!--more-->

## 原則對應錯誤
### 狀況
VPN 連線的時候，出現「原則對應錯誤」的錯誤訊息  
![原則對應錯誤](/assets/img/2019-12-02/5Y6f5YmH5bCN5oeJ6Yyv6Kqk.png)

### 解法
這是因為 Windows 預設不支援 `DH2048_AES256`  
修改登錄機碼：
在 `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters` 底下新增：  
```
名稱：`NegotiateDH2048_AES256`
類型：`REG_DWORD`
資料：`1`
```

## 明明已經設定帳號密碼，卻會說密碼錯誤，要求再輸入一次
### 狀況
新增 VPN 設定時，有設定帳號密碼上去  
但是連線的時候會要再輸入一次

### 解法
在 VPN server 安裝 plugins
```
sudo apt-get install libcharon-extra-plugins
```
(保險起見，記得重啟 VPN 服務)

## VPN 顯示已連線，但是 IP 沒有變
### 狀況
VPN 顯示已連線，但是 IP 沒有變

### 解法
調整 VPN 介面卡的設定
![預設路由](/assets/img/2019-12-02/default-route.png)

## 參考資料
<a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ikev2-vpn-server-with-strongswan-on-ubuntu-18-04-2" target="_blank" rel="noopener">https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ikev2-vpn-server-with-strongswan-on-ubuntu-18-04-2</a>  
<a href="https://www.willnet.net/index.php/archives/113/" target="_blank" rel="noopener">https://www.willnet.net/index.php/archives/113/</a>
