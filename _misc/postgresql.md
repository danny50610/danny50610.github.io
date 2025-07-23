---
layout: page
title: PostgreSQL Cheat Sheet
description: "紀錄常用 PostgreSQL 指令"
---

久久用一次，乾脆抄下來備用

# postgres user
- 切換到 postgres user
``` bash
sudo su - postgres
```

- 建立 Database
```bash
createdb my_site
```

- 建立 User
```bash
createuser my_site_user -P
```
```
Enter password for new role: 
Enter it again:
```

- List table size (order by total_bytes desc)
```sql
SELECT *, pg_size_pretty(total_bytes) AS total
    , pg_size_pretty(index_bytes) AS index
    , pg_size_pretty(toast_bytes) AS toast
    , pg_size_pretty(table_bytes) AS "table"
  FROM (
  SELECT *, total_bytes-index_bytes-coalesce(toast_bytes,0) AS table_bytes FROM (
      SELECT c.oid,nspname AS table_schema, relname AS table_name
              , c.reltuples AS row_estimate
              , pg_total_relation_size(c.oid) AS total_bytes
              , pg_indexes_size(c.oid) AS index_bytes
              , pg_total_relation_size(reltoastrelid) AS toast_bytes
          FROM pg_class c
          LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
          WHERE relkind = 'r'
  ) a
) a ORDER BY total_bytes desc;
```

- 使用 postgres user 執行 psql
``` bash
sudo su - postgres -c psql
```

# psql
- 授權 User `my_site_user` 有 database `my_site` 所有權限
```
grant all privileges on database my_site to my_site_user;
```

- List Databases
<div class="language-plaintext highlighter-rouge" style="margin-left: 30px;"><div class="highlight"><pre class="highlight"><code style="white-space: pre;">homestead=# \l
                                                      List of databases
      Name       |   Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |     Access privileges      
-----------------+-----------+----------+------------+------------+------------+-----------------+----------------------------
 my_site         | homestead | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =Tc/homestead             +
                 |           |          |            |            |            |                 | homestead=CTc/homestead   +
                 |           |          |            |            |            |                 | my_site_user=CTc/homestead
 postgres        | homestead | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0       | homestead | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/homestead              +
                 |           |          |            |            |            |                 | homestead=CTc/homestead
 template1       | homestead | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/homestead              +
                 |           |          |            |            |            |                 | homestead=CTc/homestead
</code></pre></div></div>

- connect database
```
\c my_site
```

- List Table
```
\dt
```
