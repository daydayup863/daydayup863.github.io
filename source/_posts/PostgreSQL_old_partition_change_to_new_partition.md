---
title: PostgreSQL旧触发器分区表更换为新分区表 
date: 2021-05-07 17:21:02
tags: 
- PostgreSQL
- partition table
categories: 
- PostgreSQL
- partition table
top: 49 
description: 
password: 

---

利用PostgreSQL新分区表attach功能，完成旧分区表替换成新分区表

<!--more-->

# 创建旧分区表 (仅用于测试，触发器函数就不写了)
```
mydb=# create table t_partition(id int, name text, create_time timestamptz);
CREATE TABLE
mydb=#
mydb=# create table t_old_child_2020() inherits ( t_partition);
CREATE TABLE
mydb=# \d+ t_old_child_2020
                                                Table "public.t_old_child_2020"
   Column    |           Type           | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
-------------+--------------------------+-----------+----------+---------+----------+-------------+--------------+-------------
 id          | integer                  |           |          |         | plain    |             |              |
 name        | text                     |           |          |         | extended | pglz        |              |
 create_time | timestamp with time zone |           |          |         | plain    |             |              |
Inherits: t_partition
Access method: heap

mydb=# create table t_old_child_2021() inherits ( t_partition);
CREATE TABLE
mydb=#
mydb=#
mydb=# \d+ t_partition
                                                  Table "public.t_partition"
   Column    |           Type           | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
-------------+--------------------------+-----------+----------+---------+----------+-------------+--------------+-------------
 id          | integer                  |           |          |         | plain    |             |              |
 name        | text                     |           |          |         | extended | pglz        |              |
 create_time | timestamp with time zone |           |          |         | plain    |             |              |
Child tables: t_old_child_2020,
              t_old_child_2021
Access method: heap

mydb=# alter table t_old_child_2020 add check (create_time>='2020-01-01 00:00:00' and create_time<'2021-01-01 00:00:00' );
ALTER TABLE
mydb=#
mydb=# alter table t_old_child_2021 add check (create_time>='2021-01-01 00:00:00' and create_time<'2022-01-01 00:00:00' );
ALTER TABLE
mydb=#
mydb=# insert into t_old_child_2020 select 1, 'a', '2020-03-01 00:00:00';
INSERT 0 1
mydb=# insert into t_old_child_2021 select 1, 'a', '2021-03-01 00:00:00';
INSERT 0 1
mydb=#

```

# 创建新分区表
```
mydb=# create table t_partition_new(id int, name text, create_time timestamptz) partition by range(create_time);
CREATE TABLE
mydb=# 

```


# 交换分区表
```
db=# alter table t_old_child_2020 no inherit t_partition;
ALTER TABLE
mydb=# alter table t_partition_new attach partition t_old_child_2020 for values from ('2020-01-01 00:00:00') to ('2021-01-01 00:00:00');
ALTER TABLE
mydb=# 
mydb=# alter table t_old_child_2021 no inherit t_partition;
ALTER TABLE
mydb=# 
mydb=# alter table t_partition_new attach partition t_old_child_2021 for values from ('2021-01-01 00:00:00') to ('2022-01-01 00:00:00');
ALTER TABLE
mydb=# 
mydb=# begin ;alter table t_partition rename  to t_partition_old; alter table t_partition_new rename to t_partition; commit;
BEGIN
ALTER TABLE
ALTER TABLE
COMMIT
mydb=# 

```

# 验证数据
```
mydb=# table  t_partition;
 id | name |      create_time
----+------+------------------------
  1 | a    | 2020-03-01 00:00:00+08
  1 | a    | 2021-03-01 00:00:00+08
(2 rows)

mydb=#

```
