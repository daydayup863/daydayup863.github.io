---
title: pg_hint_plan 
date: 2020-08-18 14:25:34
tags: 
- pg_hint_plan
- PostgreSQL
categories: 
- PostgreSQL
top: 16
description: 
password: 

---

# 名字

pg_hint_plan – 在注释中使用特殊格式的 hint 语句来控制查询计划.

# 语法

PostgresQL 使用查询规划器计算开销，查询规划器是基于数据统计的而不是静态的规则。对于一个 SQL 语句查询规划器估计每一种可能执行方法的开销，然后使用开销最低的执行方法。查询规划器使用它认为最好的执行计划，而并非真正最优的，因为它不考虑一些数据的属性，例如，列之间的关系。

pg_hint_plan 使用所谓的 “hint” 来调整执行计划，它可以在 SQL 注释中使用特殊的格式来简单的描述。

<!-- more -->

# 描述

## 基本用法

pg_hint 在 SQL 注释的特殊格式中读取 hint 语句。特殊格式以 "/+" 开头以 "/" 结尾。hint 语句由名字和跟在后面的括号中的参数组成，参数以空格分隔。为了可读性每一个 hint 语句使用换行来分隔。

在下面的例子中，哈希 join 被作为 join 的方法，使用顺数扫描作为扫描的方法。

```
postgres=# /*+
postgres*#    HashJoin(a b)
postgres*#    SeqScan(a)
postgres*#  */
postgres-# EXPLAIN SELECT *
postgres-#    FROM pgbench_branches b
postgres-#    JOIN pgbench_accounts a ON b.bid = a.bid
postgres-#   ORDER BY a.aid;
                                      QUERY PLAN
---------------------------------------------------------------------------------------
 Sort  (cost=31465.84..31715.84 rows=100000 width=197)
   Sort Key: a.aid
   ->  Hash Join  (cost=1.02..4016.02 rows=100000 width=197)
         Hash Cond: (a.bid = b.bid)
         ->  Seq Scan on pgbench_accounts a  (cost=0.00..2640.00 rows=100000 width=97)
         ->  Hash  (cost=1.01..1.01 rows=1 width=100)
               ->  Seq Scan on pgbench_branches b  (cost=0.00..1.01 rows=1 width=100)
(7 rows)

postgres=#
```

## hints 的类型

hint 语句通过他们影响的实体而分为四种类型。扫描方法，join 方法，join 顺序，纠正行数和 GUC 设置。对于每一种类型的hint语句有一个 hint 语句列表http://pghintplan.sourceforge.jp/hint_list.html

## hint 的扫描方法

hint 的扫描方法通过在表上指定一个参数来选择一种扫描方式。pg_hint 可以通过表的别名来识别一个表。他们可以是 ‘SeqScan’，‘IndexScan’ 等等。

hint 扫描在普通表，继承表，UNLOGGED 表，临时表，系统表是有效的。它不能应用在外部表，表函数，命令的值（VALUES command results），CTEs，视图和子查询中。

## Hints 的 join 方法

使用表名作为参数的 jion 方法名来指定 Hints 的 join 方法

在参数列表中可以使用普通表，继承表，UNLOGGED 表，临时表，外部表，系统表，表函数，命令的值（VALUES command results），CTEs 作为参数。但是视图和子查询不可以。

## Hint 的 join 顺序

可以使用 “Leading” 来指定 join 的顺序。join 的顺序将按照参数列表的给出顺序来执行。

## Hint 纠正行数

由于查询规划器的能力限制，它可能错误的估计一些条件下的结果集的数量。这种类型的 hint 将会纠正这种情况。

## GUC 参数的临时设置

在查询规划时设置 hint 来改变 GUC 的参数。在 Query Planning 指定 GUC 的参数可以在查询规划时得到预期的效果，除非其他的 hint 和查询规划器配置的参数冲突了。相同的 GUC 参数在 hint 中最后一个配置的将会生效。对于pg_hint_plan 可以通过 hint 来设置 GUC 参数，但是它不一定会按照预期的工作。详细内容可以看限制章节。

# pg_hint_plan 的 GUC 参数

下面的 GUC 参数会影响 pg_hint_plan 的行为.

| Parameter name | description | Default |
|--|--|--|
| pg_hint_plan.enable_hint | Enbles or disables the function of pg_hint_plan. | on |
| pg_hint_plan.debug_print | Enables and select the verbosity of the debug output of pg_hint_plan. off, on, detailed and verbose are valid. | off |
| pg_hint_plan.message_level | Specifies the message level of debug prints. error, warning, notice, info, log, debug are valid and fatal and panic are inhibited. | info |

对于这些 GUC 参数 PostgreSQL 9.1 需要定义一个变量类. 详细内容见 custom_variable_classes.

# 安装

这部分描述了安装的步骤。

## 编译二进制模块

在源码的根目录执行 “make”，然后使用合适的角色执行 “make install”。在这个过程中对于 PostgresQL 应该将环境变量设置为合适的值。
```
$ tar xzvf pg_hint_plan-1.x.x.tar.gz
$ cd pg_hint_plan-1.x.x
$ make
$ su
$ make install
```

## 加载 pg_hint_plan

pg_hint_plan 不需要使用 CREATE EXTENSION.只要使用 LOAD 命令将它激活，当然你也可以全局的加载它通过在 postgresql.conf 中设置 shared_preload_libraries。你也可以使用 ALTER USER SET/ALTER DATABASE SET 在指定的会话中自动的加载它。

```
postgres=# LOAD 'pg_hint_plan';
LOAD
postgres=#
```

如果你计划 hint 表，你需要设置 pg_hint_plan.enable_hint_tables 值为 on。

## 卸载

如果你使用源码安装了 pg_hint_plan，你可以在源码的根目录使用 "make uninstall" 来卸载安装的文件。

```
$ cd pg_hint_plan-1.x.x
$ su
# make uninstall
```

# Hint 描述

这部分描述了怎么写各种类型的 hints。

## 扫描方法 hints

扫描 hints 使用一个参数去指定目标对象。使用索引作为参数最好使用索引名。目标对象如果有别名参数应该指定为别名。在下面的例子中 table1 使用顺序扫描，table2 使用主键索引扫描。

```
postgres=# /*+
postgres*#     SeqScan(t1)
postgres*#     IndexScan(t2 t2_pkey)
postgres*#  */
postgres-# SELECT * FROM table1 t1 JOIN table table2 t2 ON (t1.key = t2.key);
```

## Join hints

join hints 使用两个或多个组成 join 的对象作为参数。如果指定了三个对象，hint 将会 join 两个对象后再 join 他们中的一个。在下面的例子中，首先使用嵌套循环 join talbe1 和 table2，然后使用合并 join 前面的结果和 table3.
```
postgres=# /*+
postgres*#     NestLoop(t1 t2)
postgres*#     MergeJoin(t1 t2 t3)
postgres*#     Leading(t1 t2 t3)
postgres*#  */
postgres-# SELECT * FROM table1 t1
postgres-#     JOIN table table2 t2 ON (t1.key = t2.key)
postgres-#     JOIN table table3 t3 ON (t2.key = t3.key);

```

## join 顺序

尽管先 join table2 和 table3 然后 join table1 这种情况可能出现，但是 NestLoop hint 将不会生效。"Leading" hint 在这种情况下可以强制改变 join 顺序。在上面的例子中 Leading hint 改变 join 顺序为 table1,2,3 然后这两种 join 方法都会生效。

上面 Leading hint 的形式改变了 join 的顺序，但是查询规划器的 join 顺序是自左至右的。如果你想改变 join 的方向，第二种方法是有效的。

```
postgres=# /*+ Leading((t1 (t2 t3))) */ SELECT...
```

每对括号包括两个元素，可以是对象也可以是嵌套的括号。括号中的第一个元素是驱动者或外部表，第二个是被驱动或者内部。

## hints 纠正结果集数量

如果查询规划器错误的估计了在一些条件下join返回的结果集数量。这个 hint 可以通过几种方法来纠正这个值，包括绝对值，加减和乘法。参数是组成 join 的对象和操作。下面的例子通过4个例子给出了纠正 a join b 返回的值数量的用法。

```
postgres=# /*+ Rows(a b #10) */ SELECT... ; Sets rows of join result to 10
postgres=# /*+ Rows(a b +10) */ SELECT... ; Increments row number by 10
postgres=# /*+ Rows(a b -10) */ SELECT... ; Subtracts 10 from the row number.
postgres=# /*+ Rows(a b *10) */ SELECT... ; Makes the number 10 times larger.
```

## GUC 临时设置

在目标语句设置查询规划器使用的 GUC 参数，下面的列子，在该查询中设置查询规划器使用 random_page_cost 的值为 2.0

```
postgres=# /*+
postgres*#     Set(random_page_cost 2.0)
postgres*#  */
postgres-# SELECT * FROM table1 t1 WHERE key = 'value';
...

```

# Hint 语法

## Hint 注释位置

pg_hint_plan 在第一个注释块读取 hint，在这个注释块中只允许有字母，数字，空格，下划线，逗号，和括号。在下面的例子中 HashJoin(a b) and SeqScan(a) 被认为是 hint 而 IndexScan(a) and MergeJoin(a b) 不是 hint.

```
postgres=# /*+
postgres*#    HashJoin(a b)
postgres*#    SeqScan(a)
postgres*#  */
postgres-# /*+ IndexScan(a) */
postgres-# EXPLAIN SELECT /*+ MergeJoin(a b) */ *
postgres-#    FROM pgbench_branches b
postgres-#    JOIN pgbench_accounts a ON b.bid = a.bid
postgres-#   ORDER BY a.aid;
                                      QUERY PLAN
---------------------------------------------------------------------------------------
 Sort  (cost=31465.84..31715.84 rows=100000 width=197)
   Sort Key: a.aid
   ->  Hash Join  (cost=1.02..4016.02 rows=100000 width=197)
         Hash Cond: (a.bid = b.bid)
         ->  Seq Scan on pgbench_accounts a  (cost=0.00..2640.00 rows=100000 width=97)
         ->  Hash  (cost=1.01..1.01 rows=1 width=100)
               ->  Seq Scan on pgbench_branches b  (cost=0.00..1.01 rows=1 width=100)
(7 rows)

postgres=#
```

在对象名字中转义特殊的字符

作为 hint 参数的对象如果包括括号，双引号和空格应该使用双引号。和 PostgresQL 的转义规则相同。

使用相同名字表之间的区分

同一对象使用重复的名字出现多次和在不同表空间中使用相同名字的对象可以通过使用别名来区别，并且在 hint 语句中使用这些别名。下面的例子第一个 SQL 语句因为在查询语句中使用一个表名两次而导致了错误，而第二个语句可以正常工作因为 t1 每次出现使用了不同的别名并且在 HashJoin hint 中使用了别名。

```
postgres=# /*+ HashJoin(t1 t1)*/
postgres-# EXPLAIN SELECT * FROM s1.t1
postgres-# JOIN public.t1 ON (s1.t1.id=public.t1.id);
INFO:  hint syntax error at or near "HashJoin(t1 t1)"
DETAIL:  Relation name "t1" is ambiguous.
                            QUERY PLAN
------------------------------------------------------------------
 Merge Join  (cost=337.49..781.49 rows=28800 width=8)
   Merge Cond: (s1.t1.id = public.t1.id)
   ->  Sort  (cost=168.75..174.75 rows=2400 width=4)
         Sort Key: s1.t1.id
         ->  Seq Scan on t1  (cost=0.00..34.00 rows=2400 width=4)
   ->  Sort  (cost=168.75..174.75 rows=2400 width=4)
         Sort Key: public.t1.id
         ->  Seq Scan on t1  (cost=0.00..34.00 rows=2400 width=4)
(8 行)

postgres=# /*+ HashJoin(pt st) */
postgres-# EXPLAIN SELECT * FROM s1.t1 st
postgres-# JOIN public.t1 pt ON (st.id=pt.id);
                             QUERY PLAN
---------------------------------------------------------------------
 Hash Join  (cost=64.00..1112.00 rows=28800 width=8)
   Hash Cond: (st.id = pt.id)
   ->  Seq Scan on t1 st  (cost=0.00..34.00 rows=2400 width=4)
   ->  Hash  (cost=34.00..34.00 rows=2400 width=4)
         ->  Seq Scan on t1 pt  (cost=0.00..34.00 rows=2400 width=4)
(5 行)

postgres=#

```

# 限制

## 在 from 子句中多种值的限制

无论是语法中给定别名还是在 explain 中显示的描述，所有在 from 子句中出现的值都具有相同的名字 "VALUES"。所以如果他们在目标查询中出现两次及以上将不能使用 hints。

## 在继承表上的使用

继承表不能单独的使用 hint。他们和他们父表共享相同的 hint。

## 通过设置 hint 来设置 pg_hint_plan 参数

pg_hint_plan 参数改变了它原来的行为所以一些参数不能按照期待的执行。

    hint 改变了 enalbe_hint,enable_hint_tables 被忽略了，但是他们在 debug 日志中记录为 “used hints”。
    设置 debug_print 和 message_level 工作在目标查询的中间处理。

# hint 在目标语句中使用方法

## hint 在查询语句中隐含实体的使用

Hint 对于任何带有目的名字的对象都是有效的，即使他们没有出现在查询语句中，例如在视图里的对象。这样如果你想使用不同于第一个视图的 hint 你可以在不同的视图对同一对象使用不用的别名。

在下面的例子中，在第一个查询中出现的两个表中使用了相同的名字 t1,所以 hint SeqScan(t1) 将会在两次扫描中生效。另一方面第二个语句中在这两个出现的表中使用了不同的名字 t3 所以 hint 只影响这一个扫描。

这个机制也可以应用在两个重写的查询语句中。

```
postgres=# CREATE VIEW view1 AS SELECT * FROM table1 t1;
CREATE TABLE
postgres=# /*+ SeqScan(t1) */
postgres=# EXPLAIN SELECT * FROM table1 t1 JOIN view1 t2 ON (t1.key = t2.key) WHERE t2.key = 1;
                           QUERY PLAN
-----------------------------------------------------------------
 Nested Loop  (cost=0.00..358.01 rows=1 width=16)
   ->  Seq Scan on table1 t1  (cost=0.00..179.00 rows=1 width=8)
         Filter: (key = 1)
   ->  Seq Scan on table1 t1  (cost=0.00..179.00 rows=1 width=8)
         Filter: (key = 1)
(5 rows)

postgres=# /*+ SeqScan(t3) */
postgres=# EXPLAIN SELECT * FROM table1 t3 JOIN view1 t2 ON (t1.key = t2.key) WHERE t2.key = 1;
                                   QUERY PLAN
--------------------------------------------------------------------------------
 Nested Loop  (cost=0.00..187.29 rows=1 width=16)
   ->  Seq Scan on table1 t3  (cost=0.00..179.00 rows=1 width=8)
         Filter: (key = 1)
   ->  Index Scan using foo_pkey on table1 t1  (cost=0.00..8.28 rows=1 width=8)
         Index Cond: (key = 1)
(5 rows)
```

## Hint 在继承中的使用

hint 作用在父表将自动影响它的所有孩子。子表不能自己指定自己的 hint。

## hint 在多条语句中的作用范围

一个多语句描述只能有一个 hint 注释并且这个注释会作用在这个多语句的所有单条语句上。注意在一个交互的 psql 的看似多语句实际上是一系列单条语句，所以 hint 只作用在跟在它后面的一条语句。相反每一个单条语句有他们自己的 hint 注释。

## 在一些上下文中的子查询

在下面的上下文中子查询也可以使用 hint。

IN (SELECT ... {LIMIT | OFFSET ...} ...)
= ANY (SELECT ... {LIMIT | OFFSET ...} ...)
= SOME (SELECT ... {LIMIT | OFFSET ...} ...)

对于这些语法，当计划 jion 子查询结果时，查询规划器内部分配子查询的名字为 “ANY subquery”，所以 join hint 使用隐含的名字作用在这些 join 中。

```
postgres=# /*+HashJoin(a1 ANY_subquery)*/
postgres=# EXPLAIN SELECT *
postgres=#    FROM pgbench_accounts a1
postgres=#   WHERE aid IN (SELECT bid FROM pgbench_accounts a2 LIMIT 10);
                                         QUERY PLAN

---------------------------------------------------------------------------------------------
 Hash Semi Join  (cost=0.49..2903.00 rows=1 width=97)
   Hash Cond: (a1.aid = a2.bid)
   ->  Seq Scan on pgbench_accounts a1  (cost=0.00..2640.00 rows=100000 width=97)
   ->  Hash  (cost=0.36..0.36 rows=10 width=4)
         ->  Limit  (cost=0.00..0.26 rows=10 width=4)
               ->  Seq Scan on pgbench_accounts a2  (cost=0.00..2640.00 rows=100000 width=4)
(6 rows)
```

## 使用 IndexOnlyScan hint (PostgreSQL 9.2及之后的版本)

如果你在一个表上使用 IndexOnlyScan hint 你应该明确的指定一个能执行仅扫描的索引，而其他的索引不能执行仅扫描。否则 pg_hint_plan 可能会选择他们.

## NoIndexScan hint 的预防要点 (PostgreSQL 9.2 及以后版本)

NoIndexScan hint 涉及到 NoIndexOnlyScan.

# hints 的错误处理

在大多数情况下 pg_hint_plan 停止解析任何错误并且使用 hints 已经解析的内容。下面是一些典型的错误。

## 语法错误

任何语法错误或者错误的 hint 名字被记录为语法错误。如果 pg_hint_plan.debug_print 被设置为 on 这些错误将会记录在服务器的日志中，并使用 pg_hint_plan.message_level 中指定的信息级别。

## 错误规格

对象的错误规格将会导致被 hints 忽略。这种错误竟会和语法错误一样在日志中被记录为 “not used hints”。

## 冗余或冲突的 hints

当冗余 hints 或者互相冲突的 hint 出现时最后一个 hint 将会生效。这种错误将会和语法错误一样在日志中被记录为 “duplication hints”。

## 嵌套注释

在 hint 的注释内不能嵌套另一个注释。如果 pg_hint_plan 发现这种情况，不同于其它错误的处理方法，它停止解析并且放弃已经解析的所有 hint。这种错误和其他错误以一样的方式记录。

# 函数限制

## GUC 参数对查询规划器的影响

对于 FROM 子句的数量超过了 from_collapse_limit 的情况，查询规划器不会考虑 join 的顺序。在这种情况下 pg_hint_plan 不会影响 join 的顺序。

## pg_hint_plan 本质上无效的情况

因为 pg_hint_plan 性质，它对查询规划器作用范围之外的情况是无效的，包括下面的情况：

- 使用嵌套循环的 FULL OUTER JOIN
- 没有使用索引资格的列去使用索引
- 不带 ctid 条件的查询做 TID 扫描

## 在 ECPG 程序中的查询

在嵌入式 SQL 语句中 ECPG 删除了查询语句中的注释所以 hints 不能传给这些查询语句。唯一的方法是通过命令传 给定的未修改的字符串。在这种情况下需要考虑 hint 表。

## 查询指纹的影响

相同的查询使用不同的注释在 PostgresQL 9.2及以后会产生相同的指纹，但是他们在9.1及以前会产生不同的指纹，带有不同 hint 的相同的查询在这个版本被作为单独查询。

# 英文版

http://pghintplan.sourceforge.jp/pg_hint_plan.html
