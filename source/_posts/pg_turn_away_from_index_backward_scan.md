---
title: order by + limit N index backward scan 慢SQL优化
date: 2021-03-15 14:22:23
tags: 
- sql
- limit
- order by
- slow query
categories: 
- sql
- limit
- order by
- slow query
top: 37 
description: 
password: 

---


# backward  index scan
```
mydb=# explain analyze select data_id,create_time from test_table where status=0 and city_id=310188 and type=103  and sub_type=any(array[10306,10304,10305]) order by create_time desc limit 1;
                                                                                    QUERY PLAN                                                                                    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.44..1615.86 rows=1 width=16) (actual time=10945.944..10945.945 rows=1 loops=1)
   ->  Index Scan Backward using test_table_create_time_idx on test_table  (cost=0.44..1936888.15 rows=1199 width=16) (actual time=10945.942..10945.942 rows=1 loops=1)
         Filter: ((status = 0) AND (city_id = 310188) AND (type = 103) AND (sub_type = ANY ('{10306,10304,10305}'::integer[])))
         Rows Removed by Filter: 9975856
 Planning time: 0.573 ms
 Execution time: 10945.974 ms
(6 rows) 
```

<!--more-->

# set  enable_indexscan to off;
```
mydb=# set  enable_indexscan to off;
SET
mydb=# explain analyze select data_id,create_time from test_table where status=0 and city_id=310188 and type=103  and sub_type=any(array[10306,10304,10305]) order by create_time desc limit 1;
                                                                                QUERY PLAN                                                                                
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=4593.63..4593.63 rows=1 width=16) (actual time=0.642..0.642 rows=1 loops=1)
   ->  Sort  (cost=4593.63..4596.63 rows=1199 width=16) (actual time=0.642..0.642 rows=1 loops=1)
         Sort Key: create_time
         Sort Method: top-N heapsort  Memory: 25kB
         ->  Bitmap Heap Scan on test_table  (cost=25.61..4587.64 rows=1199 width=16) (actual time=0.603..0.636 rows=15 loops=1)
               Recheck Cond: ((city_id = 310188) AND (sub_type = ANY ('{10306,10304,10305}'::integer[])) AND (type = 103) AND (status = 0))
               Heap Blocks: exact=11
               ->  Bitmap Index Scan on test_table_city_id_sub_type_create_time_idx  (cost=0.00..25.31 rows=1199 width=0) (actual time=0.581..0.581 rows=15 loops=1)
                     Index Cond: ((city_id = 310188) AND (sub_type = ANY ('{10306,10304,10305}'::integer[])))
 Planning time: 0.395 ms
 Execution time: 0.671 ms
(11 rows)
```

# with子句
```
mydb=# explain analyze with cte as(select data_id,create_time from test_table where status=0 and city_id=310188 and type=103  and sub_type=any(array[10306,10304,10305]) order by create_time desc)
mydb-# select * from cte limit 1;
                                                                                 QUERY PLAN                                                                                 
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=4651.95..4651.97 rows=1 width=16) (actual time=0.093..0.093 rows=1 loops=1)
   CTE cte
     ->  Sort  (cost=4648.95..4651.95 rows=1199 width=16) (actual time=0.090..0.090 rows=1 loops=1)
           Sort Key: test_table.create_time
           Sort Method: quicksort  Memory: 25kB
           ->  Bitmap Heap Scan on test_table  (cost=25.61..4587.64 rows=1199 width=16) (actual time=0.048..0.078 rows=15 loops=1)
                 Recheck Cond: ((city_id = 310188) AND (sub_type = ANY ('{10306,10304,10305}'::integer[])) AND (type = 103) AND (status = 0))
                 Heap Blocks: exact=11
                 ->  Bitmap Index Scan on test_table_city_id_sub_type_create_time_idx  (cost=0.00..25.31 rows=1199 width=0) (actual time=0.041..0.041 rows=15 loops=1)
                       Index Cond: ((city_id = 310188) AND (sub_type = ANY ('{10306,10304,10305}'::integer[])))
   ->  CTE Scan on cte  (cost=0.00..23.98 rows=1199 width=16) (actual time=0.092..0.092 rows=1 loops=1)
 Planning time: 0.476 ms
 Execution time: 0.132 ms
(13 rows)
```
# 增加无关排序列
```
mydb=# explain analyze with cte as(select data_id,create_time from test_table where status=0 and city_id=310188 and type=103  and sub_type=any(array[10306,10304,10305]) order by create_time desc)
select * from cte limit 1;
                                                                                       QUERY PLAN                                                                                       
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=4504.43..4504.45 rows=1 width=16) (actual time=0.387..0.387 rows=1 loops=1)
   CTE cte
     ->  Sort  (cost=4501.43..4504.43 rows=1199 width=16) (actual time=0.385..0.385 rows=1 loops=1)
           Sort Key: test_table.create_time
           Sort Method: quicksort  Memory: 25kB
           ->  Index Scan using test_table_city_id_sub_type_create_time_idx on test_table  (cost=0.44..4440.12 rows=1199 width=16) (actual time=0.138..0.351 rows=15 loops=1)
                 Index Cond: ((city_id = 310188) AND (sub_type = ANY ('{10306,10304,10305}'::integer[])))
   ->  CTE Scan on cte  (cost=0.00..23.98 rows=1199 width=16) (actual time=0.387..0.387 rows=1 loops=1)
 Planning time: 3.259 ms
 Execution time: 0.438 ms
(10 rows)

```

# 新建索引
```
create index CONCURRENTLY myindex on test_table(city_id, create_time) where  status=0  and type=103;

mydb=# explain analyze  select ctid,data_id from test_table where status=0 and city_id=310188 and type=103  and sub_type=any(array[10306,10304,10305]) order by create_time desc limit 1;
                                                                                   QUERY PLAN                                                                                   
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.44..76.86 rows=1 width=22) (actual time=1.134..1.134 rows=1 loops=1)
   ->  Index Scan Backward using test_table_city_id_create_time_idx on test_table  (cost=0.44..91628.47 rows=1199 width=22) (actual time=1.133..1.133 rows=1 loops=1)
         Index Cond: (city_id = 310188)
         Filter: (sub_type = ANY ('{10306,10304,10305}'::integer[]))
         Rows Removed by Filter: 796
 Planning time: 0.344 ms
 Execution time: 1.153 ms
(7 rows)

```



