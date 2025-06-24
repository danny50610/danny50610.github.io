---
layout: post
title:  "AWS S3 在東京(ap-northeast-1)與台北(ap-east-2)的速度比較"
date:   2025-06-17 21:27:00 +08:00
tags:   [aws, s3, taipei, tokyo]
---

最近 AWS 的台北區(ap-east-2)終於啟用了(<a href="https://aws.amazon.com/tw/events/taiwan/news/taipei-region/" target="_blank" rel="noopener">新聞</a>)  
直覺上從台北不走海底電纜會比較快(而且考量最近的情況，應該也會比較穩定)  
所以想做個簡單測試來看一下速度差異

<!--more-->

經過快速的 Google 後，決定使用 MinIO 的 <a href="https://github.com/minio/warp" target="_blank" rel="noopener">warp</a> 工具來測試  
測試地點是在台北，分別測試 GET 和 PUT 的速度

首先是 GET 的部分：

| 測試項目 | 伺服器位置 | 平均傳輸速度 (MiB/s) | 請求延遲 (平均/50%/90%/99%) (ms) | TTFB (平均/最佳/最差) (ms) |
|---|---|---|---|---|
| GET | **ap-northeast-1** | **3.71** | 平均: **52.6** / 50%: **52.2** / 90%: **55.1** / 99%: **69.3** | 平均: **53** / 最佳: **46** / 最差: **290** |
| GET | **ap-east-2** | **9.97** | 平均: **19.6** / 50%: **19.5** / 90%: **21.7** / 99%: **23.8** | 平均: **20** / 最佳: **13** / 最差: **295** |

接著是 PUT 的部分：

| 測試項目 | 伺服器位置 | 平均傳輸速度 (MiB/s) | 請求延遲 (平均/50%/90%/99%) (ms) |
|---|---|---|---|
| PUT | **ap-northeast-1** | **2.88** | 平均: **67.8** / 50%: **65.4** / 90%: **78.1** / 99%: **111.3** |
| PUT | **ap-east-2** | **6.71** | 平均: **29.1** / 50%: **28.6** / 90%: **33.1** / 99%: **46.2** |

可以看到速度快了2倍以上，而且延遲也少了一半以上

測試指令如下：  
(已移除 access-key 與 secret-key)
```
warp put --host=s3.amazonaws.com --region=ap-northeast-1 --bucket=warp-benchmark-bucket-danny-test-ap-northeast-1 --obj.size=10KiB --tls --access-key= --secret-key=
warp get --host=s3.amazonaws.com --region=ap-northeast-1 --bucket=warp-benchmark-bucket-danny-test-ap-northeast-1 --objects=100 --obj.size=10KiB --tls --access-key= --secret-key=

warp put --host=warp-benchmark-bucket-danny-test-ap-east-2.s3.dualstack.ap-east-2.amazonaws.com --region=ap-east-2 --bucket=warp-benchmark-bucket-danny-test-ap-east-2 --obj.size=10KiB --tls --access-key= --secret-key=
warp get --host=warp-benchmark-bucket-danny-test-ap-east-2.s3.dualstack.ap-east-2.amazonaws.com --region=ap-east-2 --bucket=warp-benchmark-bucket-danny-test-ap-east-2 --objects=100 --obj.size=10KiB --tls --access-key= --secret-key=
```
註：`ap-east-2` 的 `host` 參數不能填 `s3.amazonaws.com`，`region` 會變回預設值

## 2025-06-24 補充測試成本

使用 AWS 的 tag 功能，可以記錄特定 Bucket 的費用
這邊包含為了熟悉 warp 有多幾次小測試  
總共的費用是 USD 1.44  
(還好有縮小測試數量和檔案大小)
