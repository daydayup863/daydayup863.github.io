---
title: SQL正则判断字符串是不是数字
date: 2021-04-22 17:24:32
tags: 
- PostgreSQL
- regexp
categories: 
- PostgreSQL
- regexp
top: 45 
description: 
password: 

---

SQL正则判断字符串是不是数字

<!--more-->

```
test=# select '11.1.1.1' ~ '^(-?\d+)(\.\d+)?$';
 ?column?
----------
 f
(1 row)

test=# select '11.1' ~ '^(-?\d+)(\.\d+)?$';
 ?column?
----------
 t
(1 row)

test=# select '-11.1' ~ '^(-?\d+)(\.\d+)?$';
 ?column?
----------
 t
(1 row)

test=# select '-11.1a' ~ '^(-?\d+)(\.\d+)?$';
 ?column?
----------
 f
(1 row)

test=# select '-11' ~ '^(-?\d+)(\.\d+)?$';
 ?column?
----------
 t
(1 row)

test=#
```
