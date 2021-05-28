---
title: PostgreSQL 12 实现 MYSQL ON UPDATE CURRENT_TIMESTAMPTZ功能
date: 2021-03-21 14:57:29
tags: 
- PostgreSQL
- default
- on update
categories: 
- PostgreSQL
- default
- on update
top: 44 
description: 
password: 

---

PostgreSQL 12 实现 MYSQL ON UPDATE CURRENT_TIMESTAMPTZ功能

<!--more-->


```
postgres=# create or replace function im_now () returns timestamptz as $$
postgres$#   select CURRENT_TIMESTAMP;
postgres$# $$ language sql strict immutable;
CREATE FUNCTION

postgres=# create table test_generated (id int primary key, info text, crt_time timestamp, 
mod_time timestamp GENERATED ALWAYS AS (im_now()) stored);
CREATE TABLE

postgres=# table  test_generated ;
 id | info | crt_time | mod_time
----+------+----------+----------
(0 rows)

postgres=# insert into test_generated select 1;
INSERT 0 1
postgres=# table  test_generated ;
 id | info | crt_time |          mod_time
----+------+----------+----------------------------
  1 |      |          | 2021-04-21 14:54:24.886718
(1 row)

postgres=# insert into test_generated select 2;
INSERT 0 1
postgres=# table  test_generated ;
 id | info | crt_time |          mod_time
----+------+----------+----------------------------
  1 |      |          | 2021-04-21 14:54:24.886718
  2 |      |          | 2021-04-21 14:54:29.742564
(2 rows)

postgres=# update test_generated set info = 'a' where id =1;
UPDATE 1
postgres=# update test_generated set info = 'a' where id =2;
UPDATE 1
postgres=# table  test_generated ;
 id | info | crt_time |          mod_time
----+------+----------+----------------------------
  1 | a    |          | 2021-04-21 14:54:46.63875
  2 | a    |          | 2021-04-21 14:54:48.158909
(2 rows)

postgres=# \d+ test_generated
                                                         Table "public.test_generated"
  Column  |            Type             | Collation | Nullable |                Default                | Storage  | Stats target | Description
----------+-----------------------------+-----------+----------+---------------------------------------+----------+--------------+-------------
 id       | integer                     |           | not null |                                       | plain    |              |
 info     | text                        |           |          |                                       | extended |              |
 crt_time | timestamp without time zone |           |          |                                       | plain    |              |
 mod_time | timestamp without time zone |           |          | generated always as (im_now()) stored | plain    |              |
Indexes:
    "test_generated_pkey" PRIMARY KEY, btree (id)
Access method: heap

postgres=#

```
