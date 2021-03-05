---
title: 利用PostgreSQL LATERAL完成行列转换 
date: 2020-08-05
tags: 
- PostgreSQL
- 行列转换
- LATERAL
- 自连接
categories: PostgreSQL
top: 1
password: 

---

利用PostgreSQL LATERAL完成行列转换.

<!-- more -->

# 需求
```
原始表数据如下：

name| English | Physics | Math
------+---------+---------+------
Simon |      90 |      76 |   79
Lucy  |     100 |      90 |   85
Lily  |      95 |      81 |   84
David |     100 |      86 |   89

转换为
  name  | subject | score
--------+---------+-------
 Simon  | english |    90
 Simon  | physics |    76
 Simon  | math    |    79
 Lucy   | english |   100
 Lucy   | physics |    90
 Lucy   | math    |    85
 Lily   | english |    95
 Lily   | physics |    81
 Lily   | math    |    84
 David  | english |   100
 David  | physics |    86
 David  | math    |    89
```

# 测试
```
create table test(name text, english int, physics int, math int);
\copy test from stdin with delimiter '|'
 Simon  |      90 |      76 |   79
 Lucy   |     100 |      90 |   85
 Lily   |      95 |      81 |   84
 David  |     100 |      86 |   89
\.
```

# 使用union all
```
mydb=# explain analyze  select name, max(english) from test group by name union all select name, max(physics) from test group by name union all select name, max(math) from test group by name;
                                                      QUERY PLAN
----------------------------------------------------------------------------------------------------------------------
 Append  (cost=78.10..249.30 rows=600 width=36) (actual time=0.071..0.151 rows=12 loops=1)
   ->  HashAggregate  (cost=78.10..80.10 rows=200 width=36) (actual time=0.069..0.078 rows=4 loops=1)
         Group Key: test.name
         ->  Seq Scan on test  (cost=0.00..55.40 rows=4540 width=36) (actual time=0.029..0.033 rows=4 loops=1)
   ->  HashAggregate  (cost=78.10..80.10 rows=200 width=36) (actual time=0.027..0.035 rows=4 loops=1)
         Group Key: test_1.name
         ->  Seq Scan on test test_1  (cost=0.00..55.40 rows=4540 width=36) (actual time=0.009..0.012 rows=4 loops=1)
   ->  HashAggregate  (cost=78.10..80.10 rows=200 width=36) (actual time=0.022..0.029 rows=4 loops=1)
         Group Key: test_2.name
         ->  Seq Scan on test test_2  (cost=0.00..55.40 rows=4540 width=36) (actual time=0.007..0.010 rows=4 loops=1)
 Planning Time: 0.511 ms
 Execution Time: 0.378 ms
(12 rows)

mydb=#
```

# 自连接
```
mydb=# explain analyze SELECT t.name,s.* from test t JOIN LATERAL(VALUES('english',t.english ), ('physics',t.physics), ('math',t.math)) s(subject, score) on true;
                                                  QUERY PLAN
--------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=0.00..361.85 rows=13620 width=68) (actual time=0.038..0.073 rows=12 loops=1)
   ->  Seq Scan on test t  (cost=0.00..55.40 rows=4540 width=44) (actual time=0.021..0.025 rows=4 loops=1)
   ->  Values Scan on "*VALUES*"  (cost=0.00..0.04 rows=3 width=36) (actual time=0.003..0.007 rows=3 loops=4)
 Planning Time: 0.209 ms
 Execution Time: 0.128 ms
(5 rows)
```
