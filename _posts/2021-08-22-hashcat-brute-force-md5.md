---
layout: post
title:  "使用 hashcat 暴力破解 md5"
date:   2021-08-22 15:13:00 +0800
tags:   [hashcat, md5]
---

早期的網站都會直接將使用者的密碼用 md5 hash 後，再直接儲存起來  
但萬一這些 hash 值外洩的話，其實是有機會在短時間內被暴力破解的

所以近代的網站都改用 bcrypt 這類 password-hashing function 來提高暴力破解的難度

這篇文章將使用 hashcat 來嘗試暴力破解 md5 hash

<!--more-->

## md5

這次實驗的電腦設備：
```
CPU: Intel(R) Core(TM) i5-8300H
OS: Windows 10 21H1 19043.1165
顯示卡：NVIDIA GeForce GTX 1050
```

首先先準備一個 md5 hash
```
密碼：nOY41l
md5 hash：f6a050b83ab58ff79fb433fb574bb432
```
為了避免實驗跑太久，所以挑 6 位英數混和的密碼

將 hash 儲存到 `test.hash`
```
f6a050b83ab58ff79fb433fb574bb432
```

接著使用 hashcat 開始暴破
```
> hashcat.exe -a 3 -m 0 test.hash ?a?a?a?a?a?a
```
參數說明：
- `-a`: Attack-mode，3 是 Brute-force 
- `-m`: Hash-type，0 是 md5
- `?a?a?a?a?a?a`: mask，表示嘗試所有 6 位英數混合+其他特殊符號的字串  
(可以用 `hashcat --help` 查看說明)

執行結果：
```
(前面省略 ...)

f6a050b83ab58ff79fb433fb574bb432:nOY41l

Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: f6a050b83ab58ff79fb433fb574bb432
Time.Started.....: Sun Aug 22 15:39:23 2021 (53 secs)
Time.Estimated...: Sun Aug 22 15:40:16 2021 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Mask.......: ?a?a?a?a?a?a [6]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2334.1 MH/s (8.30ms) @ Accel:32 Loops:128 Thr:1024 Vec:1
Recovered........: 1/1 (100.00%) Digests
Progress.........: 123399536640/735091890625 (16.79%)
Rejected.........: 0/123399536640 (0.00%)
Restore.Point....: 13598720/81450625 (16.70%)
Restore.Sub.#1...: Salt:0 Amplifier:3968-4096 Iteration:0-128
Candidate.Engine.: Device Generator
Candidates.#1....: [*S,)% -> dIe"1l
Hardware.Mon.#1..: Temp: 75c Util: 94% Core:1721MHz Mem:3504MHz Bus:16

Started: Sun Aug 22 15:39:22 2021
Stopped: Sun Aug 22 15:40:18 2021
```

大約跑了 1 分鐘左右，可以看到上面有成功破解 hash，冒號後面就是這次使用的密碼
```
f6a050b83ab58ff79fb433fb574bb432:nOY41l
```

## md5 + salt
或許你會說，那如果加上 salt 呢？  
會提高攻擊難度嗎？

首先一樣先準備一個 md5 hash
```
密碼：nOY41l
salt：danny50610
md5 hash：83ca6dc0856a31e86b3758cd1529a7cc
```
為了避免實驗跑太久，所以挑 6 位英數混和的密碼  
hash 的產生法：md5($salt.$pass)

將 hash 和 salt 儲存到 `test.hash`
```
83ca6dc0856a31e86b3758cd1529a7cc:danny50610
```

接著使用 hashcat 開始暴破
```
> hashcat.exe -a 3 -m 20 test.hash ?a?a?a?a?a?a
```
參數說明：
- `-m`: Hash-type，20 是 md5($salt.$pass)

執行結果：
```
(前面省略 ...)

83ca6dc0856a31e86b3758cd1529a7cc:danny50610:nOY41l

Session..........: hashcat
Status...........: Cracked
Hash.Name........: md5($salt.$pass)
Hash.Target......: 83ca6dc0856a31e86b3758cd1529a7cc:danny50610
Time.Started.....: Sun Aug 22 15:51:35 2021 (1 min, 8 secs)
Time.Estimated...: Sun Aug 22 15:52:43 2021 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Mask.......: ?a?a?a?a?a?a [6]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1846.3 MH/s (4.86ms) @ Accel:8 Loops:256 Thr:1024 Vec:1
Recovered........: 1/1 (100.00%) Digests
Progress.........: 124005212160/735091890625 (16.87%)
Rejected.........: 0/124005212160 (0.00%)
Restore.Point....: 13721600/81450625 (16.85%)
Restore.Sub.#1...: Salt:0 Amplifier:3840-4096 Iteration:0-256
Candidate.Engine.: Device Generator
Candidates.#1....: q]~61l -> dIe"1l
Hardware.Mon.#1..: Temp: 70c Util: 88% Core:1733MHz Mem:3504MHz Bus:16

Started: Sun Aug 22 15:51:32 2021
Stopped: Sun Aug 22 15:52:44 2021
```

也大約跑了 1 分鐘左右，可以看到上面有成功破解 hash，最後一段就是這次使用的密碼
```
83ca6dc0856a31e86b3758cd1529a7cc:danny50610:nOY41l
```

## bcrypt
接著測試 bcrypt

首先一樣先準備一個 hash
```
密碼：nOY41l
bcrypt hash：$2y$10$r1ZvwFjz3z9i2swDyZHC5eiJGv/HRsLDR/ZJR68Lz3isZc0xCVC3u
```

將 hash 儲存到 `test.hash`
```
$2y$10$r1ZvwFjz3z9i2swDyZHC5eiJGv/HRsLDR/ZJR68Lz3isZc0xCVC3u
```

接著使用 hashcat 開始暴破
```
> hashcat.exe -a 3 -m 3200 test.hash ?a?a?a?a?a?a
```
參數說明：
- `-m`: Hash-type，3200 是 bcrypt

執行結果：
```
(前面省略 ...)

Session..........: hashcat
Status...........: Running
Hash.Name........: bcrypt $2*$, Blowfish (Unix)
Hash.Target......: $2y$10$r1ZvwFjz3z9i2swDyZHC5eiJGv/HRsLDR/ZJR68Lz3is...xCVC3u
Time.Started.....: Sun Aug 22 16:43:21 2021 (6 mins, 1 sec)
Time.Estimated...: Fri May 29 11:56:17 2178 (156 years, 279 days)
Kernel.Feature...: Pure Kernel
Guess.Mask.......: ?a?a?a?a?a?a [6]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      149 H/s (5.87ms) @ Accel:4 Loops:4 Thr:12 Vec:1
Recovered........: 0/1 (0.00%) Digests
Progress.........: 53520/735091890625 (0.00%)
Rejected.........: 0/53520 (0.00%)
Restore.Point....: 480/7737809375 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:33-34 Iteration:688-692
Candidate.Engine.: Device Generator
Candidates.#1....: zryler -> z@@ner
Hardware.Mon.#1..: Temp: 56c Util: 94% Core:1771MHz Mem:3504MHz Bus:16
```

可以看到預估時間高達 156 年，所以就不讓他跑完了

## 結論

從上面的實驗可以發現，bcrypt 比 md5 更可以抵擋暴力攻擊  
不過 bcrypt 也不是萬能的，假設攻擊者使用對 bcrypt 最佳化的 FPGA 進行攻擊，是有可能大幅加速的  
但 FPGA 跟顯示卡比起來，使用難度大上不少 

另外可以發現就算 md5 加上 salt，雖然速度降低很多，但還算是快的  
所以通常會建議使用 bcrypt 
