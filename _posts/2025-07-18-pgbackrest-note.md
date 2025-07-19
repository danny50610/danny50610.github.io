---
layout: post
title:  "pgBackRest 架設筆記"
date:   2025-07-18 19:07:00 +08:00
tags:   [pgBackRest]
---

原本是用 pg_dump 為基礎的工具幫網站備份 PostgreSQL 資料  
但是先在機器上產生檔案，再上傳到 S3 上，導致要先佔用一部分的儲存空間  
加上每次都是完整備份，S3 儲存空間使用量變得非常大

所以想要使用 pgBackRest 來備份，這邊紀錄一下架設的指令與設定檔

<!--more-->

選用 pgBackRest 有以下幾點考量：
- 可以直接上傳到 S3
- 支援加密
- 支援 Full, Differential, & Incremental 備份

## 安裝
因為機器 OS 是用 Ubuntu，且 APT 有加上 <a href="https://www.postgresql.org/download/linux/ubuntu/" target="_blank">apt.postgresql.org</a>  
所以直接：
```bash
sudo apt install pgbackrest
```

驗證安裝：
```bash
$ sudo -u postgres pgbackrest
pgBackRest 2.55.1 - General help

Usage:
    pgbackrest [options] [command]

Commands:
    annotate        add or modify backup annotation
    archive-get     get a WAL segment from the archive
    archive-push    push a WAL segment to the archive
    backup          backup a database cluster
    check           check the configuration
    expire          expire backups that exceed retention
    help            get help
    info            retrieve information about backups
    repo-get        get a file from a repository
    repo-ls         list files in a repository
    restore         restore a database cluster
    server          pgBackRest server
    server-ping     ping pgBackRest server
    stanza-create   create the required stanza data
    stanza-delete   delete a stanza
    stanza-upgrade  upgrade a stanza
    start           allow pgBackRest processes to run
    stop            stop pgBackRest processes from running
    verify          verify contents of a repository
    version         get version

Use 'pgbackrest help [command]' for more information.
```

## 設定
### S3
建立 Bucket, Access Key 和 Secret Key

### pgBackRest
設定 `/etc/pgbackrest.conf`，S3 的部分要換成剛剛設定的
```
[global]
repo1-type=s3
repo1-s3-bucket=xxxxx-bucket
repo1-s3-endpoint=xxxxx-endpoint
repo1-s3-key=xxxxx
repo1-s3-key-secret=xxxxx
repo1-s3-region=us-east-1
repo1-retention-full=28
repo1-retention-full-type=time
repo1-cipher-pass=xxxxx
repo1-cipher-type=aes-256-cbc

start-fast=y
delta=y
archive-async=y
archive-push-queue-max=100MB

[global:archive-push]
compress-level=3

[main]
pg1-path=/var/lib/postgresql/15/main
```

#### 參數說明

設定完整備份至少保留 28 天 (`retention-full-type=time`)
```
repo1-retention-full=28
repo1-retention-full-type=time
```

如果要加密的話，可以設定這兩個參數，密碼可以使用 `openssl rand -base64 48` 產生
```
repo1-cipher-pass=xxxxx
repo1-cipher-type=aes-256-cbc
```

如果是使用 Minio 來替代 S3，要記得調整設定
```
repo1-s3-uri-style=path
```

#### 初始化 stanza
這邊會在 S3 上建立必要的檔案，可以剛好驗證連線  
如果有變更 stanza 的名稱，要記得替換 `--stanza` 參數
```bash
$ sudo -u postgres pgbackrest --stanza=main --log-level-console=info stanza-create
```

範例輸出：
```
2025-07-16 06:42:45.472 P00   INFO: stanza-create command begin 2.55.1: --exec-id=890770-b94b6b8a --log-level-console=info --pg1-path=/var/lib/postgresql/15/main --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-s3-bucket=xxxx --repo1-s3-endpoint=xxxxx --repo1-s3-key=<redacted> --repo1-s3-key-secret=<redacted> --repo1-s3-region=us-east-1 --repo1-type=s3 --stanza=main
2025-07-16 06:42:45.481 P00   INFO: stanza-create for stanza 'main' on repo1
2025-07-16 06:42:47.016 P00   INFO: stanza-create command end: completed successfully (1548ms)
```

### PostgreSQL

設定 `/etc/postgresql/15/main/postgresql.conf`，並重新啟動
```
archive_mode = on
archive_command = 'pgbackrest --stanza=main archive-push %p'
wal_level = replica
```

持續備份 WAL 是必要的，不然還原出來的資料庫會無法使用

接著執行 `check` 指令確認設定
```bash
$ sudo -u postgres pgbackrest --stanza=main --log-level-console=info check
```

範例輸出：
```
2025-07-16 08:16:44.161 P00   INFO: check command begin 2.55.1: --exec-id=894728-775125c7 --log-level-console=info --pg1-path=/var/lib/postgresql/15/main --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-s3-bucket=xxxx --repo1-s3-endpoint=xxxxx --repo1-s3-key=<redacted> --repo1-s3-key-secret=<redacted> --repo1-s3-region=us-east-1 --repo1-type=s3 --stanza=main
2025-07-16 08:16:44.193 P00   INFO: check repo1 configuration (primary)
2025-07-16 08:16:44.631 P00   INFO: check repo1 archive for WAL (primary)
2025-07-16 08:16:46.977 P00   INFO: WAL segment 000000010000000D00000008 successfully archived to '/var/lib/pgbackrest/archive/main/15-1/000000010000000D/000000010000000D00000008-3c6485e7ed9dec8f0f964d55bdfba092fd815fd1.gz' on repo1
2025-07-16 08:16:46.978 P00   INFO: check command end: completed successfully (2822ms)
```

### Backup
預設是 Full 備份
```bash
$ sudo -u postgres pgbackrest --stanza=main --log-level-console=info backup
```

範例輸出：
```
2025-07-16 08:22:05.655 P00   INFO: backup command begin 2.55.1: --checksum-page --delta --exec-id=894932-75be644c --log-level-console=info --pg1-path=/var/lib/postgresql/15/main --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-retention-full=28 --repo1-retention-full-type=time --repo1-s3-bucket=xxxx --repo1-s3-endpoint=xxxxx --repo1-s3-key=<redacted> --repo1-s3-key-secret=<redacted> --repo1-s3-region=us-east-1 --repo1-type=s3 --stanza=main --start-fast
2025-07-16 08:22:06.004 P00   WARN: checksum-page option set to true but checksums are not enabled on the cluster, resetting to false
2025-07-16 08:22:06.081 P00   WARN: no prior backup exists, incr backup has been changed to full
2025-07-16 08:22:06.081 P00   INFO: execute non-exclusive backup start: backup begins after the requested immediate checkpoint completes
2025-07-16 08:22:06.121 P00   INFO: backup start archive = 000000010000000D0000000A, lsn = D/A000028
2025-07-16 08:22:06.121 P00   INFO: check archive for prior segment 000000010000000D00000009
2025-07-16 08:40:44.330 P00   INFO: execute non-exclusive backup stop and wait for all WAL segments to archive
2025-07-16 08:40:44.466 P00   INFO: backup stop archive = 000000010000000D0000000A, lsn = D/A043E68
2025-07-16 08:40:44.895 P00   INFO: check archive for segment(s) 000000010000000D0000000A:000000010000000D0000000A
2025-07-16 08:40:45.985 P00   INFO: new backup label = 20250716-082206F
2025-07-16 08:40:49.470 P00   INFO: full backup size = 3.8GB, file total = 2801
2025-07-16 08:40:49.470 P00   INFO: backup command end: completed successfully (1123824ms)
2025-07-16 08:40:49.471 P00   INFO: expire command begin 2.55.1: --exec-id=894932-75be644c --log-level-console=info --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-retention-full=28 --repo1-retention-full-type=time --repo1-s3-bucket=xxxx --repo1-s3-endpoint=xxxxx --repo1-s3-key=<redacted> --repo1-s3-key-secret=<redacted> --repo1-s3-region=us-east-1 --repo1-type=s3 --stanza=main
2025-07-16 08:40:49.733 P00   INFO: repo1: time-based archive retention not met - archive logs will not be expired
2025-07-16 08:40:49.733 P00   INFO: expire command end: completed successfully (263ms)
```

Differential 備份
```bash
$ sudo -u postgres pgbackrest --stanza=main --log-level-console=info --type=diff backup
```

Incremental 備份
```bash
$ sudo -u postgres pgbackrest --stanza=main --log-level-console=info --type=incr backup
```

### Restore

沒有經過 Restore 測試的備份不是備份

首先要先停止 postgresql
```bash
$ sudo systemctl stop postgresql@15-main.service
```

**整個資料庫會被覆蓋**，Restore 前要確認清楚
```bash
$ sudo -u postgres pgbackrest --stanza=main --log-level-console=info restore
```

範例輸出：
```
2025-07-18 10:55:37.334 P00   INFO: restore command begin 2.55.1: --delta --exec-id=7903-3d1b5292 --log-level-console=info --pg1-path=/var/lib/postgresql/15/main --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-s3-bucket=xxxx --repo1-s3-endpoint=xxxxx --repo1-s3-key=<redacted> --repo1-s3-key-secret=<redacted> --repo1-s3-region=us-east-1 --repo1-type=s3 --stanza=main
2025-07-18 10:55:37.414 P00   INFO: repo1: restore backup set 20250716-082206F_20250716-183022I, recovery will start at 2025-07-16 18:30:22
2025-07-18 10:55:37.415 P00   INFO: remove invalid files/links/paths from '/var/lib/postgresql/15/main'
2025-07-18 10:56:29.513 P00   INFO: write updated /var/lib/postgresql/15/main/postgresql.auto.conf
2025-07-18 10:56:29.515 P00   INFO: restore global/pg_control (performed last to ensure aborted restores cannot be started)
2025-07-18 10:56:29.516 P00   INFO: restore size = 3.8GB, file total = 2801
2025-07-18 10:56:29.517 P00   INFO: restore command end: completed successfully (52184ms)
```

完成後啟動資料庫驗證資料
```bash
$ sudo systemctl start postgresql@15-main.service
```

### Cron
``` bash
$ crontab -e -u postgres
```

```
# m h  dom mon dow   command
30 19  *   *   0     pgbackrest --type=full --stanza=main backup > /dev/null 2>&1
30 19  *   *   1-6   pgbackrest --type=diff --stanza=main backup > /dev/null 2>&1
```

## 監控備份與 WAL 狀況

// TODO: 待補充

## 參考資料
* <a href="https://pgbackrest.org/" target="_blank">pgBackRest Home page</a>
* <a href="https://pgstef.github.io/2023/07/03/pgbackrest_differential_vs_incremental_backup.html" target="_blank">pgBackRest differential vs incremental backup</a>
* <a href="https://dataegret.com/2025/02/avoiding-the-wal-archives-retention-trap-in-pgbackrest/" target="_blank">Avoiding the WAL Archives Retention Trap in pgBackRest</a>
* <a href="https://medium.com/pgsql-tw/pgbackrest-%E5%85%A7%E5%BE%AA%E7%92%B0%E6%8C%81%E7%BA%8C%E5%82%99%E4%BB%BD-22fae06a1a61" target="_blank">pgBackRest 內循環持續備份</a>
* <a href="https://bootvar.com/guide-to-setup-pgbackrest/" target="_blank">Step-by-Step Guide to setup pgBackRest</a>




