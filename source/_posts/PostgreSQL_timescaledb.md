---
title:  TimescaleDB 使用记录
date: 2021-04-23 16:18:08
tags: 
- PostgreSQL
- TimescaleDB
categories: 
- PostgreSQL
- TimescaleDB
top: 45 
description: 
password: 

---

记录timescaledb使用过程中遇到的一些问题及解决方案

<!--more-->


# 用函数的方式重启TimescaleDB Background Worker Scheduler, 不影响业务的正常使用

```
postgres@jintao-ThinkPad-L490:~/timescaledb-1.7.5/sql$ psql
psql (12.3)
Type "help" for help.

postgres=# CREATE OR REPLACE FUNCTION _timescaledb_internal.restart_background_workers()
RETURNS BOOL
AS '/home/postgres/pg12/lib/timescaledb', 'ts_bgw_db_workers_restart'
LANGUAGE C VOLATILE;
CREATE FUNCTION
postgres=#

postgres@jintao-ThinkPad-L490:~/pg12/lib$ ps -ef|grep "TimescaleDB Background Worker Scheduler"
postgres 2641542 2634972  0 16:11 ?        00:00:00 postgres: TimescaleDB Background Worker Scheduler
postgres 2641572 2544080  0 16:12 pts/9    00:00:00 grep --color=auto TimescaleDB Background Worker Scheduler
postgres@jintao-ThinkPad-L490:~/pg12/lib$
postgres@jintao-ThinkPad-L490:~/pg12/lib$
postgres@jintao-ThinkPad-L490:~/pg12/lib$ psql -c "select _timescaledb_internal.restart_background_workers();"
 restart_background_workers
----------------------------
 t
(1 row)

postgres@jintao-ThinkPad-L490:~/pg12/lib$ ps -ef|grep "TimescaleDB Background Worker Scheduler"
postgres 2641592 2634972  1 16:12 ?        00:00:00 postgres: TimescaleDB Background Worker Scheduler
postgres 2641594 2544080  0 16:12 pts/9    00:00:00 grep --color=auto TimescaleDB Background Worker Scheduler
postgres@jintao-ThinkPad-L490:~/pg12/lib$

```

