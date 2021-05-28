---
title: PostgreSQL14 允许配置TOAST压缩方法
date: 2021-05-12 10:52:31
tags: 
- COMPRESSION
- POstgreSQL
- lz4
- pglz
categories: 
- COMPRESSION
- POstgreSQL
- lz4
- pglz
top: 50 
description: 
password: 

---

PostgreSQL 14 允许配置TOAST压缩方法, 默认为pglz, 可以通过GUC default_toast_compression 设置默认值，当前支持pglz, lz4两种配置, 支持lz4需要在configure时增加--with-lz4.

<!--more-->

# 查看当前default_toast_compression

```
postgres=# show default_toast_compression;
 default_toast_compression
---------------------------
 pglz
(1 row)

postgres=#
```


# 设置toast压缩方法
```
mydb=# CREATE TABLE test_compression (f1 TEXT COMPRESSION pglz, f2 TEXT COMPRESSION lz4);
CREATE TABLE
mydb=#
mydb=# \d+ test_compression
                                   Table "public.test_compression"
 Column | Type | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
--------+------+-----------+----------+---------+----------+-------------+--------------+-------------
 f1     | text |           |          |         | extended | pglz        |              |
 f2     | text |           |          |         | extended | lz4         |              |
Access method: heap

mydb=#
mydb=# alter table test_compression alter COLUMN f1 set compression lz4;
ALTER TABLE
mydb=# alter table test_compression alter COLUMN f2 set compression pglz;
ALTER TABLE
mydb=#
mydb=# \d+ test_compression
                                   Table "public.test_compression"
 Column | Type | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
--------+------+-----------+----------+---------+----------+-------------+--------------+-------------
 f1     | text |           |          |         | extended | lz4         |              |
 f2     | text |           |          |         | extended | pglz        |              |
Access method: heap

mydb=#

```

# 测试
```
mydb=# insert into test_compression select repeat('a', 204800) , repeat('a', 204800);
INSERT 0 1
mydb=#
mydb=#
mydb=# insert into test_compression select repeat('a', 2048000) , repeat('a', 2048000);
INSERT 0 1
mydb=#
mydb=# select pg_column_compression(f1),
       pg_column_size(f1),
       pg_column_compression(f2),
       pg_column_size(f2),
       pg_column_size(f2)/pg_column_size(f1)::numeric from test_compression
;
 pg_column_compression | pg_column_size | pg_column_compression | pg_column_size |      ?column?
-----------------------+----------------+-----------------------+----------------+--------------------
 lz4                   |            822 | pglz                  |           2356 | 2.8661800486618005
 lz4                   |           8050 | pglz                  |          23449 | 2.9129192546583851
(2 rows)

mydb=#
mydb=# \timing
Timing is on.
mydb=#
mydb=# create table test_pglz(f1 text compression pglz);
CREATE TABLE
Time: 7.706 ms
mydb=#
mydb=# create table test_lz4(f1 text compression lz4);
CREATE TABLE
Time: 5.601 ms
mydb=#
mydb=# insert into test_pglz select repeat('a', 204800) from generate_series(1, 10000);
INSERT 0 10000
Time: 39956.824 ms (00:39.957)
mydb=#
mydb=# insert into test_lz4 select repeat('a', 204800) from generate_series(1, 10000);
INSERT 0 10000
Time: 523.663 ms
mydb=#
mydb=# insert into test_pglz select repeat('a', 204800) from generate_series(1, 10000);
INSERT 0 10000
Time: 34295.272 ms (00:34.295)
mydb=#
mydb=# insert into test_lz4 select repeat('a', 204800) from generate_series(1, 10000);
INSERT 0 10000
Time: 527.297 ms
mydb=#
mydb=# insert into test_pglz select repeat('a', 204800) from generate_series(1, 10000);
INSERT 0 10000
Time: 33774.475 ms (00:33.774)
mydb=#
mydb=# insert into test_lz4 select repeat('a', 204800) from generate_series(1, 10000);
INSERT 0 10000
Time: 588.888 ms
mydb=#
mydb=# select pg_size_pretty(pg_table_size('test_lz4')) size_of_lz4, pg_size_pretty(pg_table_size('test_pglz')) size_of_pglz;
 size_of_lz4 | size_of_pglz
-------------+--------------
 25 MB       | 118 MB
(1 row)

Time: 0.548 ms
mydb=#

```

好家伙, lz4相比pglz来说, 对于压缩比高的数据, 不仅空间占用更小, 效率更是大大提高。

