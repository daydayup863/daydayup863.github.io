---
title: Linux ubuntu postgresql 源码搭建主从 
date: 2020-08-11 12:04:03
tags: 
- postgres
- pg
- install
- source
- postgresql
- PostgreSQL
categories:
- PostgreSQL
top: 12
description: 
password: 

---

# 搭建主节点PostgreSQL实例
## 下载

[官网下载地址](https://www.postgresql.org/ftp/source/v12.3/)

```
sudo groupadd postgres
sudo useradd -a -G postgres postgres
su - postgres
wget https://ftp.postgresql.org/pub/source/v12.3/postgresql-12.3.tar.gz
```
<!-- more -->

## 解压 && 编译 && 安装

需要带哪些编译选项看个人需求
```
tar -zxvf postgresql-12.3.tar.gz
cd postgresql-12.3
./configure CFLAGS=-O0 -g' '--prefix=/home/postgres/pg12' '--with-perl' '--with-libxml' '--with-libxslt' '--with-ossp-uuid' '--with-blocksize=32' '--with-segsize=2' '--with-wal-blocksize=64' '--with-llvm' --with-python
make -j10 world       # 带插件一起编译
make install-world    # 带插件一起安装
```
make install-world 出现"PostgreSQL, contrib, and documentation installation complete."时， 即为成功


## 配置环境变量

vim ~/.bashrc 追加如下内容
```
export PGHOME=/home/postgres/pg12
export PGDATA=/home/postgres/pg120_data
export PGUSER=postgres
export PGPORT=5432
export PGDATABASE=postgres
export LD_LIBRARY_PATH=/home/postgres/pg12/lib:$LD_LIBRARY_PATH
export PATH=/home/postgres/pg12/bin:$PATH
```
重新打开窗口，或者执行 . ~/.bashrc 使之生效

## 初始化
```
postgres@pgserver1:~/postgresql-12.3$ initdb
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locales
  COLLATE:  en_US.UTF-8
  CTYPE:    en_US.UTF-8
  MESSAGES: en_US.UTF-8
  MONETARY: zh_CN.UTF-8
  NUMERIC:  zh_CN.UTF-8
  TIME:     zh_CN.UTF-8
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /home/postgres/pg120_data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Asia/Shanghai
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    pg_ctl -D /home/postgres/pg120_data -l logfile start

```

## 修改参数, 记录数据库日志
```
wal_level = replica             # minimal, replica, or logical
                                # (change requires restart)

listen_addresses = '*'          # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
port = 5432                             # (change requires restart)

logging_collector = on          # Enable capturing of stderr and csvlog
                                        # into log files. Required to be on for
                                        # csvlogs.
                                        # (change requires restart)

# These are only used if logging_collector is on:
log_directory = 'log'                   # directory where log files are written,
                                        # can be absolute or relative to PGDATA
log_filename = 'postgresql-%a.log'      # log file name pattern,
                                        # can include strftime() escapes
#log_file_mode = 0600                   # creation mode for log files,
                                        # begin with 0 to use octal notation
log_truncate_on_rotation = on           # If on, an existing log file with the
                                        # same name as the new log file will be
                                        # truncated rather than appended to.
                                        # But such truncation only occurs on
                                        # time-driven rotation, not on restarts
                                        # or size-driven rotation.  Default is
                                        # off, meaning append to existing files
```

## 启动数据库
```
postgres@pgserver1:~/postgresql-12.3$ pg_ctl start
waiting for server to start....2020-08-11 14:39:44.004 CST [3608866] LOG:  starting PostgreSQL 12.3 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.3.0-10ubuntu2) 9.3.0, 64-bit
2020-08-11 14:39:44.004 CST [3608866] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2020-08-11 14:39:44.004 CST [3608866] LOG:  listening on IPv6 address "::", port 5432
2020-08-11 14:39:44.006 CST [3608866] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2020-08-11 14:39:44.012 CST [3608866] LOG:  redirecting log output to logging collector process
2020-08-11 14:39:44.012 CST [3608866] HINT:  Future log output will appear in directory "log".
 done
server started
```

# 搭建主节点PostgreSQL实例
## 下载 

** 步骤同1.1 **

## 解压 && 编译 && 安装 

** 步骤同1.2 **

## 配置环境变量

** 步骤同1.3 **

## 主节点创建流复制用户
```
postgres@pgserver1:~/postgresql-12.3$ psql
psql (12.3)
Type "help" for help.

postgres=# \! uuidgen
7089bae5-c467-492a-a402-aaba388e6826
postgres=# create user replicator Replication password '7089bae5-c467-492a-a402-aaba388e6826';
CREATE ROLE
postgres=#
```

## 增加用户replicator ACL
主库 vim $PGDATA/pg_hba.conf 新增, x.x.x.x为从节点IP
```
host    replication     replicator      x.x.x.x/32            trust
```

使之生效
```
postgres@pgserver1:~$ psql 
psql (12.3)
Type "help" for help.

postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)
```
## 使用pg_basebackup生成从库数据目录
```
postgres@pgserver2:~$ pg_basebackup -D /home/postgres/pg120_data -R -X stream -c fast -P -h pgserver1 -p 5432 -Ureplicator
65984/65984 kB (100%), 1/1 tablespace
```
## 启动从库

```
postgres@pgserver2:~$ pg_ctl -D /home/postgres/pg120_data start
waiting for server to start....2020-08-14 15:12:31.552 CST [817489] LOG:  starting PostgreSQL 12.3 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.3.0-10ubuntu2) 9.3.0, 64-bit
2020-08-14 15:12:31.552 CST [817489] LOG:  listening on IPv4 address "x.x.x.x", port 5432
2020-08-14 15:12:31.555 CST [817489] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2020-08-14 15:12:31.566 CST [817489] LOG:  redirecting log output to logging collector process
2020-08-14 15:12:31.566 CST [817489] HINT:  Future log output will appear in directory "log".
 done
server started
```

## 主节点确认流复制
```
postgres@pgserver1:~$ psql 
psql (12.3)
Type "help" for help.
postgres=# select * from pg_stat_replication ;
  pid   | usesysid | usename  | application_name | client_addr | client_hostname | client_port |         backend_start         | backend_xmin |   state   | sent_lsn  | write_lsn | flush_lsn | replay_lsn |   writ
e_lag    |    flush_lag    |   replay_lag    | sync_priority | sync_state |          reply_time           
--------+----------+----------+------------------+-------------+-----------------+-------------+-------------------------------+--------------+-----------+-----------+-----------+-----------+------------+-------
---------+-----------------+-----------------+---------------+------------+-------------------------------
 817497 |       10 | replicator | walreceiver      | x.x.x.x   |                 |       50850 | 2020-08-14 15:12:31.612261+08 |              | streaming | 0/4000B60 | 0/4000B60 | 0/4000B60 | 0/4000B60  | 00:00:
00.02204 | 00:00:00.022362 | 00:00:00.022769 |             0 | async      | 2020-08-14 15:12:31.636213+08
(1 row)

postgres=# 
```

## 从节点确认流复制
```
postgres@pgserver2:~$ psql
psql (12.3)
Type "help" for help.

postgres=# select * from pg_stat_wal_receiver ;
  pid   |  status   | receive_start_lsn | receive_start_tli | received_lsn | received_tli |      last_msg_send_time       |     last_msg_receipt_time     | latest_end_lsn |        latest_end_time        | slot_n
ame | sender_host | sender_port |                                                                                                             conninfo                                                             
                                                
--------+-----------+-------------------+-------------------+--------------+--------------+-------------------------------+-------------------------------+----------------+-------------------------------+-------
----+-------------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------
 817496 | streaming | 0/4000000         |                 1 | 0/4000B60    |            1 | 2020-08-14 15:15:01.924922+08 | 2020-08-14 15:15:01.924989+08 | 0/4000B60      | 2020-08-14 15:12:31.613519+08 |       
    | x.x.x.x   |        5432 | user=replicator passfile=/home/postgres/.pgpass dbname=replication host=x.x.x.x port=5432 fallback_application_name=walreceiver sslmode=disable sslcompression=0 gssencmode=disab
le krbsrvname=replicator target_session_attrs=any
(1 row)

postgres=# 
```

# 结束

主从搭建so easy.

# FAQ

[PostgreSQL：编译安装常见问题](https://postgres.fun/20131118144309.html)
[RedHat Enterprise 5上安装 Postgresql](https://postgres.fun/20100731115100.html)
