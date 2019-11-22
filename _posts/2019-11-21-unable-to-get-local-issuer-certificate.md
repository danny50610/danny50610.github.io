---
layout: post
title:  "Fix: cURL error 60: SSL certificate problem: unable to get local issuer certificate"
date:   2019-11-21 23:46:00 +0800
tags:   [curl, certificate]
---

前幾個月有寫一個爬蟲，每天會從國家電影中心檢查有沒有新的票房資訊  
但是從最近幾天開始，會收到下面的錯誤訊息
```
cURL error 60: SSL certificate problem: unable to get local issuer certificate (see https://curl.haxx.se/libcurl/c/libcurl-errors.html)
```

<!--more-->

## 調查
這個爬蟲是用 PHP 寫的，但我猜底層是使用 `curl`  
先直接用 `curl` 測測看
```bash
$ curl https://www.tfi.org.tw/BoxOfficeBulletin/weekly
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```
看來不行，所以問題不在 PHP

簡單 Google 後，解決方法千百種，但是比較奇怪的是：Chrome 不認為憑證有問題  
決定使用先 <a href="https://www.ssllabs.com/ssltest/index.html" target="_blank" rel="noopener">SSL Server Test</a> 測試看看

![certification-path](/assets/img/2019-11-21/certification-path.png)

(寫文章時才截圖，結果居然修好了...)  
上圖紅色框框裡的橘線那邊，原本是 `Extra download`  
`Extra download`代表需要額外下載憑證

## 安裝憑證
使用右上角藍色框框的下載按鈕，會有整個 Path 所有用到的3個憑證  
只複製第二個憑證來安裝

這邊 Server 使用 Ubuntu 18.04，其他 OS 需要查對應的做法

到 `/usr/local/share/ca-certificates` 新增剛剛下載的憑證，副檔名用 `.crt`

結果
```bash
root@localhost:/usr/local/share/ca-certificates# cat Chunghwa-Telecom-Co.crt
-----BEGIN CERTIFICATE-----
MIIF+TCCA+GgAwIBAgIQFDWW8kQacWeYP/yVl0GbUzANBgkqhkiG9w0BAQsFADBeMQswCQYDVQQG
EwJUVzEjMCEGA1UECgwaQ2h1bmdod2EgVGVsZWNvbSBDby4sIEx0ZC4xKjAoBgNVBAsMIWVQS0kg
Um9vdCBDZXJ0aWZpY2F0aW9uIEF1dGhvcml0eTAeFw0xNDEyMTEwODUxNTlaFw0zNDEyMTEwODUx
NTlaMGAxCzAJBgNVBAYTAlRXMSMwIQYDVQQKDBpDaHVuZ2h3YSBUZWxlY29tIENvLiwgTHRkLjEs
MCoGA1UECwwjUHVibGljIENlcnRpZmljYXRpb24gQXV0aG9yaXR5IC0gRzIwggEiMA0GCSqGSIb3
DQEBAQUAA4IBDwAwggEKAoIBAQDoZb9Rajtfyy5ggweoejMEaTKKALXa8s/SO/k6NaH2b55gz3SB
MazyWWn8rXUndm//t0HJibE/mWJzbUKhmMxtoOPCL5oGTdkxKsNEcIqgSnUQIhc4HRuTfVcLNS76
MYE6HpLlDzGX5BfVqIdfze+Y/T2Mm7xBvtxu48SS9poV79BsP7w7hIUAo7gLBhnovNLKaKFci/bn
ok24R/un7PKH5H1UlhCvhsSyuMvMCL7pkean0CYO+ucTIZ7CobzuzpGthtxlt9rSR9XpzHKZ7HSr
+/Dz/i+UG6eQ5ppFs+gPIQQZAKBuWtCNwKW+6KEfJ+kIz4YqJP/YVpLeG0RV57g5AgMBAAGjggGv
MIIBqzAfBgNVHSMEGDAWgBQeDPe2Z/LhkiYJRcBVOS53P0JKojAdBgNVHQ4EFgQUy4N9ZRWvqcnz
qKn0ZHx5UgV0QGEwDgYDVR0PAQH/BAQDAgEGMEAGA1UdHwQ5MDcwNaAzoDGGL2h0dHA6Ly9lY2Eu
aGluZXQubmV0L3JlcG9zaXRvcnkvQ1JMX1NIQTIvQ0EuY3JsMIGLBggrBgEFBQcBAQR/MH0wRAYI
KwYBBQUHMAKGOGh0dHA6Ly9lY2EuaGluZXQubmV0L3JlcG9zaXRvcnkvQ2VydHMvSXNzdWVkVG9U
aGlzQ0EucDdiMDUGCCsGAQUFBzABhilodHRwOi8vb2NzcC5lY2EuaGluZXQubmV0L09DU1Avb2Nz
cEcxc2hhMjASBgNVHRMBAf8ECDAGAQH/AgEAMHUGA1UdIARuMGwwDQYLKwYBBAGBtyNkAAEwDQYL
KwYBBAGBtyNkAAIwDQYLKwYBBAGBtyNkAAMwCQYHYIZ2AWQAATAJBgdghnYBZAACMAkGB2CGdgFk
AAMwCAYGZ4EMAQIBMAgGBmeBDAECAjAIBgZngQwBAgMwDQYJKoZIhvcNAQELBQADggIBAIfkDpU4
Yj1QLp4CTeI1b5Q5xKYsUdaBx1V/VD+F+KV6flV5mNbiHQ3KA337GC4vQqJBBHM15eaaaPCgci0Y
nNhi68wWrdetQ9E1zCFZw/HCsnJrdKNgAWjCPe/4tK/AcJyztnMVeU3I+A3ApYVysfu9HhA6V7uA
h1AVyrv6Ivlui2tZamgOgpMd3qirzm4s3LCPTttJ0Q5oAIrxdw+lxPm8/Ef/yR2Dkq2dcrbPO6Hd
O3Lwbn0/WtZMeWlfU+WuoixM59yCPg72Y/RqAGI2neC3Na3QM+0g08OcBQGtSjQfdyGsa8egWnsD
NRunYmTZ13RwVRWKXXeCWFjI+URbErttSTO8sf4mXbIoyCz1JfVfZzVaZMBN7y0td5JWz1M+L9FM
u/cWnuqWPiGc20C11LGjNfG5ejqc15TE7WOeWDPTR1MopXM35I4NZKVlxfSR53g87DD1kvjtwxjA
GOoqZzpXOoe1GakMRUA2tBd6FG6GttbEl2QZK1w/t/zMlCfpdlTyG7zOcz/lKbn4GEPGwx5FPUPW
VpVIZgeeFn5MWru7ts2nvuFULQzP3Cfl94o9IkNHlXEfX0P/vXYjQnsOt3tr70+4ww9yOx3zBcvU
NpYnYXDwRLVTe4UDo++xEwimY7U7uEg3ukwCK+mlnsnabQYKflkSTf/EqonpTlEbhZve
-----END CERTIFICATE-----
```

接著執行 `sudo update-ca-certificates` 更新憑證，就正常啦！！！

## 備註
`Extra download` 這個狀況應該是 Server 那邊要修正的  
但是這個方法可用於對方不想修正，或是不想等太久的狀況

## 參考資料
* <a href="https://stackoverflow.com/questions/56547867/how-to-solve-ssl-certificate-problem-unable-to-get-local-issuer-certificate-i" target="_blank" rel="noopener">https://stackoverflow.com/questions/56547867/how-to-solve-ssl-certificate-problem-unable-to-get-local-issuer-certificate-i</a>
* <a href="https://askubuntu.com/questions/645818/how-to-install-certificates-for-command-line" target="_blank" rel="noopener">https://askubuntu.com/questions/645818/how-to-install-certificates-for-command-line</a>
