---
layout: post
title:  "修改 MC (Minio Client) 來支援 AWS 新開的台北區(ap-east-2)"
date:   2026-02-16 20:43:00 +08:00
tags:   [aws, s3, taipei, minio, mc, ap-east-2]
---

前陣子把在東京區(ap-northeast-1)的 S3 搬到台北區(ap-east-2)後  
接著要來修改 local 用來備份的腳本  
主要是使用 `mc` 將 S3 的檔案同步到 `minio` 上  
結果因為台北區(ap-east-2)是最近的成立的， `mc` 認不得  
會往錯誤的區域打 API

<!--more-->

本來想說過一陣子 `minio` 那邊就會更新了吧  
結果沒想到 `minio` 2025/12 就不在維護社群版了...

所以到 2026/02 現在這個問題還是存在  
想說趁放假來試著修修看，說不定只是哪邊有個 List 補進去就好

Clone 下來後，也把 vscode 的 Debug 設定好  
追了一下 code，開始覺得有點累，然後看到右上角的 Claude Icon  
想說之前儲值的錢還有剩，不然就叫 Claude 幫我查查看  

結果看他搜尋 code 之後，又找了一下 github 上的資料  
最後跟我說升級相依性就好了！

升級相依性也讓他處理，最後居然成功了，完全不用人工介入  
開始感受到 AI 的厲害(恐怖)  
(不過這樣花費 $1.28 (包含後續問他打包的部分)，貴不貴就看人)

最後修正好的版本在這裡：
[https://github.com/danny50610/mc/releases/tag/RELEASE-DANNY50610-1.2026-02-16T09-17-54Z](https://github.com/danny50610/mc/releases/tag/RELEASE-DANNY50610-1.2026-02-16T09-17-54Z)