---
layout: post
title:  "Certbot 自動呼叫 Nginx reload"
date:   2021-05-12 19:27:00 +0800
tags:   [certbot, nginx, Let's Encrypt]
---

最近發現 Nginx 使用到過期的 TLS 憑證  
主因是因為 Certbot 簽發新的憑證後，沒有呼叫 Nginx 進行 reload  
來載入新的憑證，導致 Nginx 繼續用舊憑證

<!--more-->

本來是有設定自動 reload 的，但設定錯誤，以及沒檢查是否能正常運作    
這邊紀錄一下設定的方式，以及如何檢查

首先，在 `/etc/letsencrypt/renewal-hooks/post` 底下新增 `01-nginx-reload` 檔案  
內容是：
```bash
#!/bin/sh
systemctl reload nginx
```
記得幫檔案加上 x 權限 (`chmod +x 01-nginx-reload`)

接著執行 `certbot renew --dry-run` 確認能正常 renew 憑證  
`--dry-run` 是讓 certbot 不要真的替換憑證  
(以下使用的主機有兩個網站)

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/**********.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator webroot, Installer None
Renewing an existing certificate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed without reload, fullchain is
/etc/letsencrypt/live/**********/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/**********.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator webroot, Installer None
Renewing an existing certificate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed without reload, fullchain is
/etc/letsencrypt/live/**********/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/**********/fullchain.pem (success)
  /etc/letsencrypt/live/**********/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Running post-hook command: /etc/letsencrypt/renewal-hooks/post/01-nginx-reload
```

可以看到最後一行有 `Running post-hook command`，路徑是剛剛建立的 `01-nginx-reload`
