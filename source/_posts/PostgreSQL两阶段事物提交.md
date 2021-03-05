---
title: 两阶段提交事物 
date: 2020-08-05 11:35:20
tags: [ PostgreSQL, 2pc ]
categories: 
- PostgreSQL
top: 10
password: 

---

# 什么是两阶段提交事物
两阶段提交协议的目标在于为分布式系统保证数据的一致性，许多分布式系统采用该协议提供对分布式事务的支持。 
顾名思义，该协议将一个分布式的事务过程拆分成两个阶段： 准备和事务提交


<!-- more -->

# PostgreSQL两阶段提交协议

    两阶段提交协议有五个步骤，如下：

1. 应用程序先调用各台机数据库做一些操作，但不提交事务。然后应用程序调用事务协调器（这个协调器可能也是由应用自己实现）中的提交方法。
2. 事务协调器将联络事务中涉及的每台数据库，并通知它们准备提交事务，这是第一阶段的开始。在PostgreSQL一般是调用“PREPARE TRANSACTION”命令。
3. 各台数据库接收到“PREPARE TRANSACTION”命令后，如果要返回成功，则数据库必须将自己置于以下状态：确保能在被要求提交事务时提交事务，或在被要求回滚事务时回滚事务。所以PostgreSQL会将已准备好提交的信息写入持久存储区中。如果数据库无法完成此事务，它会直接返回失败给事务协调器。
4. 事务协调器接收到了所有数据库的响应。
5. 在第二阶段，如果任一数据库在第一阶段返回失败，则事务协调器会将发一个回滚命令（ROLLBACK PREPARED）给各台数据库。如果所有数据库的响应都是成功的，则向各台数据库发送“COMMIT PREPARED”命令，通知各台数据库事务成功。

# 测试
```
mydb=# create table t1(id int, crt_time timestamptz);
CREATE TABLE
mydb=# begin;                                              -- 开启事务
BEGIN
mydb=*# insert into t1 select 1, now();    -- 插入一条数据
INSERT 0 1
mydb=*# prepare transaction 'p1';            -- 准备事务
PREPARE TRANSACTION
mydb=# \q
jintao@jintao-ThinkPad-L490:~$ sudo -iu jintao /opt/pg-master/bin/pg_ctl -D /export/pgdata-master/ restart -l /tmp/start.log     -- 重启数据库
waiting for server to shut down.... done
server stopped
waiting for server to start.... done
server started
jintao@jintao-ThinkPad-L490:~$ psql
psql (14devel)
Type "help" for help.

mydb=# select * from pg_prepared_xacts ;          -- 查看两阶段事务系统表
 transaction | gid |           prepared            | owner  | database
-------------+-----+-------------------------------+--------+----------
        1045 | p1  | 2020-07-15 17:02:07.180102+08 | jintao | mydb
(1 row)

mydb=# select * from t1;
 id | crt_time
----+----------
(0 rows)

mydb=# commit prepared 'p1';                            -- 提交
COMMIT PREPARED
mydb=# select * from t1;
 id |           crt_time
----+-------------------------------
  1 | 2020-07-15 17:01:59.163772+08
(1 row)

```
