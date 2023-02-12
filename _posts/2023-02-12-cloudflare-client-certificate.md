---
layout: post
title:  "Cloudflare Client Certificate 設定筆記"
date:   2023-02-12 16:14:00 +0800
tags:   [cloudflare, client-certificate]
---

最近剛好在架設新網站，為了確保只有特定電腦才能存取網站  
決定使用 Cloudflare Client Certificate 來限制

這樣憑證都可以接給 Cloudflare 管理，不過設定上實在是遇到很多問題，在這邊紀錄一下

(下方給的範例，`domain` 請換成實際使用的 domain)

<!--more-->

# 產生 Windows 可以用的格式 & 安裝

在 Cloudflare 後台新增 Client Certificate 時，會產生兩個檔案  
分別是「`-----BEGIN CERTIFICATE-----`」和「`-----BEGIN PRIVATE KEY-----`」開頭的  
這邊分別儲存成 `domain.pem` 和 `domain.key`

然後使用 openssl 建立 `domain.p12`，這邊我有設定一組密碼 (等等匯入會用到)
```shell
openssl pkcs12 -export -out domain.p12 -inkey domain.key -in domain.pem
```
(參考資料: [https://jarrodnix.dev/blog/securing-a-site-with-a-cloudflare-client-certificate-and-mtls](https://jarrodnix.dev/blog/securing-a-site-with-a-cloudflare-client-certificate-and-mtls))

接著點擊 `domain.p12` 將憑證匯入電腦

存放位置選擇「目前使用者」
{% include image.html url="/assets/img/2023-02-12/install-cert-step-1.png" class="w-50" %}

檔案名稱會自動帶入，直接下一步
{% include image.html url="/assets/img/2023-02-12/install-cert-step-2.png" class="w-50" %}

不要勾選下圖紅色框框的選項，避免被其他人匯出到其他電腦  
如果剛剛在 openssl 時有設定密碼，要在這邊輸入
{% include image.html url="/assets/img/2023-02-12/install-cert-step-3.png" class="w-50" %}

存放位置選「個人」
{% include image.html url="/assets/img/2023-02-12/install-cert-step-4.png" class="w-50" %}

可以使用 `certmgr.msc` 確認是否有安裝成功

{% include image.html url="/assets/img/2023-02-12/certmgr.png" %}

安裝後，可能會需要重開瀏覽器

# Cloudflare Web Application Firewall 設定

Cloudflare 文件給的範例不會阻擋過期的憑證，需要改用文字編輯模式手動輸入下列內容
```
(http.host in {"domain"}) and (not cf.tls_client_auth.cert_verified or cf.tls_client_auth.cert_revoked)
```

# 讓 Chrome 自動選擇憑證

Chrome 預設不會像 Firefox 記憶要使用哪一個 Client Certificate  
Windows 的話要透過修改機碼來新增政策，可以寫一個 `.reg` 檔來修改

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome\AutoSelectCertificateForUrls]
"1"="{\"pattern\":\"https://domain/\",\"filter\":{\"ISSUER\":{\"CN\":\"Managed CA ......\"}}}"

```

`CN` 的部分直接複製簽發者的名稱
{% include image.html url="/assets/img/2023-02-12/cert-cn.png" class="w-50" %}

(參考資料: [https://jarrodnix.dev/blog/securing-a-site-with-a-cloudflare-client-certificate-and-mtls](https://jarrodnix.dev/blog/securing-a-site-with-a-cloudflare-client-certificate-and-mtls))

要確定 Chrome 有沒有使用設定，可以到 chrome://policy/ 查詢，那邊也可以重新載入政策

題外話，新增政策後，Chrome 會出現「你的瀏覽器受到管理」的提示

{% include image.html url="/assets/img/2023-02-12/chrome-managed.png" class="w-50" %}
