---
title: PostgreSQL 游标获取多行 
date: 2021-04-20 17:45:17
tags:
- PostgreSQL
- refcursor
- cursor
categories: 
- PostgreSQL
- refcursor
- cursor
top: 43 
description: 
password: 

---

函数/匿名块/存储过程中游标返回多行数据.

<!--more-->


# 准备
```
postgres=# create table t(id int);
CREATE TABLE
postgres=#
postgres=# insert into t select generate_series(1,100);
INSERT 0 100
postgres=#

```


# 匿名块
```
postgres=# do
$$
declare cur cursor for select id from  t; row record;
begin
  open cur;
  for row in execute 'fetch 10 from cur'
  loop
      raise notice '%', row.id;
  end loop;

  for row in execute 'fetch 10 from cur'
  loop
      raise notice '%', row.id;
  end loop;

  close cur;

end;
$$language plpgsql;
NOTICE:  1
NOTICE:  2
NOTICE:  3
NOTICE:  4
NOTICE:  5
NOTICE:  6
NOTICE:  7
NOTICE:  8
NOTICE:  9
NOTICE:  10
NOTICE:  11
NOTICE:  12
NOTICE:  13
NOTICE:  14
NOTICE:  15
NOTICE:  16
NOTICE:  17
NOTICE:  18
NOTICE:  19
NOTICE:  20
DO
postgres=#
```

# 函数
```
postgres=# create or replace function test_func() returns setof record as
$$
declare cur cursor for select id from  t; row record;
begin
open cur; MOVE FORWARD 3 FROM cur;
  for row in execute 'fetch 10 from cur'
  loop
      raise notice '%', row.id;
  end loop;

  for row in execute 'fetch 10 from cur'
  loop
      raise notice '%', row.id;
  end loop;

  close cur;

end;
$$language plpgsql;
CREATE FUNCTION
postgres=# \e
CREATE FUNCTION
postgres=#
postgres=#
postgres=# select test_func();
NOTICE:  4
NOTICE:  5
NOTICE:  6
NOTICE:  7
NOTICE:  8
NOTICE:  9
NOTICE:  10
NOTICE:  11
NOTICE:  12
NOTICE:  13
NOTICE:  14
NOTICE:  15
NOTICE:  16
NOTICE:  17
NOTICE:  18
NOTICE:  19
NOTICE:  20
NOTICE:  21
NOTICE:  22
NOTICE:  23
 test_func
-----------
(0 rows)

postgres=#

```

```
postgres=# create or replace function test_func()
returns setof record
as
$$
declare
  cur cursor for select id from  t;
  row record;
begin
  open cur;

  MOVE FORWARD 3 FROM cur;

  return query execute 'fetch 10 from cur';

  close cur;

end;
$$language plpgsql;
CREATE FUNCTION

postgres=# select id from  test_func() as t(id int);
 id
----
  4
  5
  6
  7
  8
  9
 10
 11
 12
 13
(10 rows)

postgres=#
```
