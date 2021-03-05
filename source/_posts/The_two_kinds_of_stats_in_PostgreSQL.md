---
title: PostgreSQL的两种统计信息 
date: 2020-08-17 14:16:10
tags: 
- PostgreSQL
- stat
categories: 
- PostgreSQL
top: 15
description: 
password: 

---

时不时地，有人对重置PostgreSQL中的统计信息, 以及重置统计信息对执行计划和数据库其他部分的影响感到困惑。也许文档对此描述的会更清晰一些，但是对于那些以前从未在PostgreSQL中处理过统计信息的人来说无疑是令人困惑的。但这不仅是关于新手的问题-我在9.3中为该区域编写了一个补丁，而且我有时也会感到困惑。对于大多数用户而言，最令人惊讶的事实是“统计信息”实际上可能意味着两件事, 描述数据分布的统计信息，以及监视统计信息，跟踪有关数据库系统本身操作的计数器。每种都有不同的用途，存储方式也不同，丢失数据时的影响也大不相同。因此，让我们看看这两种统计数据的目的是什么，常见的问题是什么，当数据由于某种原因丢失时会发生什么。
<!-- more -->

# 数据分布统计信息

第一种统计信息跟踪数据的分布-不同值的数量，列中最常见的值，数据的直方图等。
这是规划查询时使用的信息, 用来解决以下问题:
    - 有多少行符合条件？ （选择性估计）
    - 联接产生多少行？ （选择性估计）
    - 聚合需要多少内存？

本质上，这些统计信息，决定了planner/optimizer使用了那一种最好的方式去执行。这种统计信息是由ANALYZE（或autovacuum）收集的，并存储在“常规”表中，并像常规数据一样受事务日志保护。检查pg_statistic系统目录，或者查看pg_stats，它是pg_statistic之上的视图，使统计信息更易于阅读。很多情况下会引发统计信息不准确，导致选择了较差的计划和糟糕的查询性能。发生这种情况的原因有多种（错误的统计信息，复杂的条件，相关的列...）

据我所知，没有命令/函数可以重置此类统计信息，因为完全没有必要，不觉得可以通过删除此类统计信息来解决该任何问题。当然尝试使用简单的DELETE从目录中删除数据（但我从未尝试过）。

# 监控统计信息

另一种统计信息是被statistics collector进程收集的，文档中的第一段
```
    PostgreSQL's statistics collector is a subsystem that supports collection and reporting of information about server activity. 
Presently, the collector can count accesses to tables and indexes in both disk-block and individual-row terms. It also tracks the 
total number of rows in each table, and information about vacuum and analyze actions for each table. It can also count calls to 
user-defined functions and the total time spent in each one.
```

因此，当您需要了解特定表的访问频率，是顺序读取表还是使用索引访问表等时，这就是统计信息收集器收集的统计信息。如果要查看此类统计信息，则有很多系统视图：

```
    pg_stat_activity
    pg_stat_archiver
    pg_stat_bgwriter
    pg_stat_database
    pg_stat_all_tables
    pg_stat_sys_tables
    ...
    pg_statio_all_tables
    pg_statio_sys_tables
    pg_statio_user_tables
    pg_statio_all_indexes
    pg_statio_sys_indexes
    ...
```

命名方案很明显（以pg_stat_或pg_statio_开头），其名称很不言而喻。例如，当您需要有关当前用户拥有的表的信息时，您可以转到pg_stat_user_tables或pg_statio_user_tables，这取决于您是否需要简单的统计信息（seq扫描，索引扫描的数量...）还是与IO相关的统计信息（数量读取的块数，点击率等）。对于与复制和数据库系统其他部分有关的其他对象（索引，函数等）和统计信息，也是如此。

# 总结
## 数据分布统计信息
-  描述数据分布的统计数据，由ANALYZE / autovacuum收集
-  planner/optimizer在规划查询时使用
-  存储在数据库中，受WAL保护的常规表
-  没有官方方法可以重置此类统计信息
-  这些问题通常会导致选择错误的查询计划

## 监控统计信息
-  统计信息跟踪数据库系统本身的运行情况
-  用于监视目的和autovacuum（以识别需要维护的对象)
-  存储在数据库外部，二进制文件（pgstat.stat）或每个数据库文件的集合中（从9.3开始）
-  这就是使用pg_stat_reset()重置的统计信息
-  最常见的问题是pgstat.stat文件变大时（由于跟踪许多数据库对象), I/O负载较高
-  重置统计信息不是解决方案-不会解决问题，并且会对autovacuum产生负面影响
-  9.3之前的解决方案：将pgstat.stat文件移动到tmpfs文件系统，考虑升级到9.3
