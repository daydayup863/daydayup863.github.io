---
title: PostgreSQL用户权限 
date: 2020-08-05 16:41:08
tags: 
- PostgreSQL
- role
- 权限
categories: PostgreSQL
top: 8
password: 

---

PostgreSQL中role是权限的集合，没有区分用户和角色的概念，"CREATE USER" 为 "CREATE ROLE" 的别名，这两个命令几乎是完全相同的,唯一的区别是"CREATE USER" 命令创建的用户默认带有LOGIN属性，而"CREATE ROLE" 命令创建的用户默认不带LOGIN属性(CREATE USER is equivalent to CREATE ROLE except that CREATE USER assumes LOGIN by default, while CREATE ROLE does not)


为了方便用role的方式管理用户， 而不是每新建一个用户就授予权限一次.

<!-- more -->

#  准备
收回PUBLIC用户组对模式public的所有权限， 并创建测试用表

```
jintao@jintao-ThinkPad-L490:~$ psql 
psql (14devel)
Type "help" for help.

mydb=# revoke ALL on SCHEMA public from PUBLIC;
REVOKE
mydb=# create table test1(id int);
CREATE TABLE
mydb=# insert into test1 select 1;
INSERT 0 1
mydb=# select * from test1;
 id 
----
  1
(1 row)

mydb=# 

```
#  创建只读角色  
  
只对table, sequence, function 做了处理， 如type, procedure等类似
```
mydb=# create role readonly;                                                 
CREATE ROLE
mydb=# grant SELECT on ALL tables in schema public to readonly ;    -- 赋予readonly对public模式下当前存在的表可读
GRANT
mydb=# grant EXECUTE on ALL functions in schema public to readonly ;    -- 赋予readonly对public模式下当前函数可执行
GRANT
mydb=# grant SELECT on ALL sequences in schema public to readonly ;      -- 赋予readonly对public模式下当前序列的可读
GRANT
mydb=# alter default privileges for role postgres in schema public grant select on tables to readonly;   -- 赋予readonly对public模式下之后对postgres用户新建的表可读
ALTER DEFAULT PRIVILEGES
mydb=# alter default privileges for role postgres  in schema public grant execute on functions to readonly;   -- 赋予readonly对public模式下之后对postgres用户新建的函数可执行
ALTER DEFAULT PRIVILEGES
mydb=# alter default privileges for role postgres  in schema public grant select on sequences to readonly;    - 赋予readonly对public模式下之后对postgres用户新建的序列可读
ALTER DEFAULT PRIVILEGES
```
#  创建读写角色  
  
只对table, sequence, function 做了处理， 如type, procedure等类似
```
mydb=# grant all on SCHEMA public to readwrite ;            -- 赋予readwrite对public模式下当前存在的表可读写
GRANT
mydb=# grant ALL  on ALL tables in schema public to readwrite ;
GRANT
mydb=# grant ALL on ALL sequences in schema public to readwrite ;
GRANT
mydb=# grant ALL on ALL functions in schema public to readwrite ;
GRANT
mydb=# alter default privileges for role postgres  in schema public grant all on tables to readwrite;
ALTER DEFAULT PRIVILEGES
mydb=# alter default privileges for role postgres in schema public grant all on sequences to readwrite;
ALTER DEFAULT PRIVILEGES
mydb=# alter default privileges for role postgres in schema public grant all on functions to readwrite;
ALTER DEFAULT PRIVILEGES

```
#  readonly 权限测试
```
mydb=# create user user1;
CREATE ROLE
mydb=# set role user1;
SET
mydb=> select * from public.test1;
ERROR:  permission denied for schema public
LINE 1: select * from public.test1;
                      ^
mydb=> insert into public.test1 select 2;
ERROR:  permission denied for schema public
LINE 1: insert into public.test1 select 2;
                    ^
mydb=> set role postgres;
SET
mydb=# grant readonly to user1 ;
GRANT ROLE
mydb=# set role user1;
SET
mydb=> select * from public.test1;
 id 
----
  1
(1 row)

mydb=> insert into public.test1 select 2;
ERROR:  permission denied for table test1
mydb=> 
mydb=# select 2 into test2;
SELECT 1
mydb=# set role user1;
SET
mydb=> select * from test2;
 ?column? 
----------
        2
(1 row)

mydb=> insert into test2 select 3;
ERROR:  permission denied for table test2
mydb=> 

```
#   readwrite权限测试
```
mydb=# create user user2;
CREATE ROLE
mydb=# set role user2;
SET
mydb=> select * from public.test1;
ERROR:  permission denied for schema public
LINE 1: select * from public.test1;
                      ^
mydb=> insert into public.test2 select 2;
ERROR:  permission denied for schema public
LINE 1: insert into public.test2 select 2;
                    ^
mydb=> set role postgres;
SET
mydb=# grant readwrite to user2;
GRANT ROLE
mydb=# set role user2;
SET
mydb=> select * from public.test1;
 id 
----
  1
(1 row)

mydb=> insert into public.test2 select 2;
INSERT 0 1
mydb=> set role postgres;
SET
mydb=# create table test3(id int);
CREATE TABLE
mydb=# set role user2;
SET
mydb=> select * from test3;
 id 
----
(0 rows)

mydb=> insert into test3 select 1;
INSERT 0 1
mydb=> 

```
