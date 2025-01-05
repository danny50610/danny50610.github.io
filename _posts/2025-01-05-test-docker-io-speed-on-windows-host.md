---
layout: post
title:  "測試 Docker 在 Windows 上的 IO 效能"
date:   2025-01-05 23:49:00 +08:00
tags:   [docker, windows]
---

由於 Laravel 網站在 Windows 上 Docker 網頁開啟速度實在太慢 (大約 3 s)  
決定嘗試看看搬到 Docker 裡面看看，就不要再把 Windows 的檔案 mount 進 Docker 的 Container 裡了  
不過在搬移前先測試看看 IO 效能差異

<!--more-->

測試方法參考這篇：<a href="https://cloud.google.com/compute/docs/disks/benchmarking-pd-performance-linux" target="_blank" rel="noopener">https://cloud.google.com/compute/docs/disks/benchmarking-pd-performance-linux</a>

這邊只測試 Rand Read ，因為對於 PHP 來說，會需要讀取多個檔案執行  
(題外話，所以在正式環境中，放進 OPCache 來降低 IO 數可以增加速度)

指令如下：
```
sudo fio --name=read_iops --directory=./io-test --size=4G \
    --time_based --runtime=5m --ramp_time=2s --ioengine=libaio --direct=1 \
    --verify=0 --bs=4K --iodepth=256 --rw=randread --group_reporting=1 \
    --iodepth_batch_submit=256  --iodepth_batch_complete_max=256
```

這次環境：
- 在 Container 中
- 在 Volume 中
- 在 Mount 進來的資料夾中

測試結果：

在 Container 中：
```
read_iops: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=256
fio-3.28
Starting 1 process
read_iops: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [r(1)][100.0%][r=619MiB/s][r=158k IOPS][eta 00m:00s]
read_iops: (groupid=0, jobs=1): err= 0: pid=290: Sun Jan  5 14:11:12 2025
  read: IOPS=157k, BW=613MiB/s (643MB/s)(180GiB/300002msec)
    slat (nsec): min=871, max=2525.9k, avg=20836.07, stdev=43453.94
    clat (usec): min=3, max=34894, avg=1606.46, stdev=149.80
     lat (usec): min=126, max=34906, avg=1627.30, stdev=140.37
    clat percentiles (usec):
     |  1.00th=[ 1287],  5.00th=[ 1467], 10.00th=[ 1483], 20.00th=[ 1516],
     | 30.00th=[ 1532], 40.00th=[ 1549], 50.00th=[ 1565], 60.00th=[ 1598],
     | 70.00th=[ 1663], 80.00th=[ 1713], 90.00th=[ 1762], 95.00th=[ 1827],
     | 99.00th=[ 2073], 99.50th=[ 2180], 99.90th=[ 2540], 99.95th=[ 2671],
     | 99.99th=[ 3163]
   bw (  KiB/s): min=545408, max=647422, per=100.00%, avg=627687.21, stdev=14714.86, samples=600
   iops        : min=136352, max=161855, avg=156921.79, stdev=3678.70, samples=600
  lat (usec)   : 4=0.01%, 10=0.01%, 20=0.01%, 50=0.01%, 100=0.01%
  lat (usec)   : 250=0.01%, 500=0.03%, 750=0.07%, 1000=0.17%
  lat (msec)   : 2=98.20%, 4=1.50%, 10=0.01%, 20=0.01%, 50=0.01%
  cpu          : usr=11.26%, sys=35.23%, ctx=3164208, majf=0, minf=59
  IO depths    : 1=0.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=44.4%, 8=36.1%, 16=17.6%, 32=1.8%, 64=0.1%, >=64=0.1%
     complete  : 0=0.0%, 4=43.2%, 8=36.5%, 16=18.2%, 32=1.9%, 64=0.1%, >=64=0.1%
     issued rwts: total=47060103,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=256

Run status group 0 (all jobs):
   READ: bw=613MiB/s (643MB/s), 613MiB/s-613MiB/s (643MB/s-643MB/s), io=180GiB (193GB), run=300002-300002msec
```

在 Volume 中：
```
read_iops: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=256
fio-3.28
Starting 1 process
read_iops: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [r(1)][100.0%][r=612MiB/s][r=157k IOPS][eta 00m:00s]
read_iops: (groupid=0, jobs=1): err= 0: pid=335: Sun Jan  5 14:19:26 2025
  read: IOPS=157k, BW=614MiB/s (643MB/s)(180GiB/300002msec)
    slat (nsec): min=840, max=3453.7k, avg=17970.30, stdev=34645.50
    clat (usec): min=4, max=33860, avg=1607.30, stdev=148.38
     lat (usec): min=104, max=33878, avg=1625.27, stdev=141.93
    clat percentiles (usec):
     |  1.00th=[ 1319],  5.00th=[ 1467], 10.00th=[ 1500], 20.00th=[ 1516],
     | 30.00th=[ 1532], 40.00th=[ 1549], 50.00th=[ 1565], 60.00th=[ 1598],
     | 70.00th=[ 1663], 80.00th=[ 1713], 90.00th=[ 1762], 95.00th=[ 1811],
     | 99.00th=[ 2024], 99.50th=[ 2147], 99.90th=[ 2507], 99.95th=[ 2671],
     | 99.99th=[ 3195]
   bw (  KiB/s): min=538180, max=645824, per=100.00%, avg=628839.59, stdev=12789.89, samples=599
   iops        : min=134545, max=161456, avg=157209.84, stdev=3197.48, samples=599
  lat (usec)   : 10=0.01%, 20=0.01%, 50=0.01%, 100=0.01%, 250=0.01%
  lat (usec)   : 500=0.02%, 750=0.05%, 1000=0.13%
  lat (msec)   : 2=98.61%, 4=1.17%, 10=0.01%, 20=0.01%, 50=0.01%
  cpu          : usr=11.14%, sys=32.44%, ctx=3339734, majf=0, minf=64
  IO depths    : 1=0.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=45.0%, 8=36.4%, 16=16.5%, 32=1.9%, 64=0.1%, >=64=0.1%
     complete  : 0=0.0%, 4=43.9%, 8=36.9%, 16=17.0%, 32=2.0%, 64=0.1%, >=64=0.1%
     issued rwts: total=47127444,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=256

Run status group 0 (all jobs):
   READ: bw=614MiB/s (643MB/s), 614MiB/s-614MiB/s (643MB/s-643MB/s), io=180GiB (193GB), run=300002-300002msec

Disk stats (read/write):
  sdg: ios=47314582/9, merge=41481/36, ticks=75902102/14, in_queue=75902118, util=100.00%
```

在 Mount 進來的資料夾中：
```
read_iops: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=256
fio-3.28
Starting 1 process
read_iops: Laying out IO file (1 file / 4096MiB)
Jobs: 1 (f=1): [r(1)][100.0%][r=16.0MiB/s][r=4100 IOPS][eta 00m:00s]
read_iops: (groupid=0, jobs=1): err= 0: pid=367: Sun Jan  5 14:30:42 2025
  read: IOPS=3610, BW=14.1MiB/s (14.8MB/s)(4233MiB/300056msec)
    slat (msec): min=35, max=251, avg=70.81, stdev=15.02
    clat (usec): min=4, max=186, avg=31.58, stdev=10.32
     lat (msec): min=35, max=251, avg=70.85, stdev=15.03
    clat percentiles (usec):
     |  1.00th=[   22],  5.00th=[   24], 10.00th=[   25], 20.00th=[   26],
     | 30.00th=[   27], 40.00th=[   29], 50.00th=[   30], 60.00th=[   32],
     | 70.00th=[   34], 80.00th=[   36], 90.00th=[   41], 95.00th=[   46],
     | 99.00th=[   70], 99.50th=[   89], 99.90th=[  137], 99.95th=[  176],
     | 99.99th=[  186]
   bw (  KiB/s): min= 6156, max=20521, per=100.00%, avg=14452.92, stdev=1902.79, samples=600
   iops        : min= 1539, max= 5130, avg=3613.23, stdev=475.69, samples=600
  lat (usec)   : 10=0.37%, 20=0.02%, 50=96.22%, 100=3.06%, 250=0.33%
  cpu          : usr=0.20%, sys=1.88%, ctx=1083397, majf=0, minf=58
  IO depths    : 1=0.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=100.0%
     submit    : 0=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=100.0%
     complete  : 0=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=100.0%
     issued rwts: total=1083392,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=256

Run status group 0 (all jobs):
   READ: bw=14.1MiB/s (14.8MB/s), 14.1MiB/s-14.1MiB/s (14.8MB/s-14.8MB/s), io=4233MiB (4439MB), run=300056-300056msec
```

結論：  
可以看到速度差異極大 (643MB/s 對比 14.8MB/s)，IOPS 也差異極大  
實際執行網頁也只需要 200ms (對比之前的 3s)  
所以推薦把 Code 放進 Volume 中就好，這樣也不怕資料不小心誤砍

題外話，那 VSCode 要如何連進去開發呢？  
原本想說是不是要在 Container 中安裝 SSH Server 呢？

後來發現其實只要在 VSCode 中安裝 "<a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers" target="_blank" rel="noopener">Dev Containers</a>"、"<a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack" target="_blank" rel="noopener">Remote Development</a>"  
然後在 VSCode 執行指令(View > Command Palette)：`Dev Containers: Attach to Running Container...`  
然後選擇主要執行的 Container 即可 (<a href="https://code.visualstudio.com/docs/devcontainers/attach-container" target="_blank" rel="noopener">參考資料</a>)
