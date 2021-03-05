---
title: psql显示风格设置
date: 2020-08-05 17:11:24
tags: psql
categories: PostgreSQL
top: 
description: " "
password: 

---

# MySQL风格
```
\pset border 2
 
postgres=# select * from test;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
+----+----+
(2 rows)
 
postgres=#
 
--------------------------------------
 
mysql> select * from t;
+------+
| id   |
+------+
|    1 |
+------+
1 row in set (0.01 sec)
 
mysql>
```
