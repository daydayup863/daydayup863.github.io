---
title: PostgreSQL 13 新特性汇总
date: 2021-04-30 11:08:32
tags: 
- PostgreSQL
- release note
categories: 
- PostgreSQL
- release note
top: 49
description: 
password: 

---


PostgreSQL 13 Beta1版本已于2020-05-21发行，目前最新版为Beta2，尽管 PostgreSQL 13 版本没有像10、11、12版本新增重量级的功能，但13版本依然增加了不少新特性和功能提升，值得细细研究。

PostgreSQL 13 典型变化如下:

逻辑复制支持分区表
Btree索引优化(引入Deduplication技术)
增量排序(Incremental Sorting)
并行VACUUM索引
并行Reindexdb
手册新增术语(Glossary)附录

本文从新特性、性能提升、数据库管理、其它亮点四方面详细介绍 PostgreSQL 13的变化。

<!--more-->

# 新特性
```
    PostgreSQL 13: 逻辑复制支持分区表
    PostgreSQL 13: CREATE SUBSCRIPTION新增publish_via_partition_root选项支持异构分区表间的数据逻辑复制
    PostgreSQL 13: 新增内置函数Gen_random_uuid()生成UUID数据
```

# 性能提升
```
    PostgreSQL 13: Btree索引优化(引入Deduplication技术)
    PostgreSQL 13: 支持增量排序(Incremental Sorting)
```

# 数据库管理
```
    PostgreSQL 13: EXPLAIN、Auto_explain、autovacuum、pg_stat_statements跟踪WAL使用信息
    PostgreSQL 13: 新增log_min_duration_sample参数控制日志记录的慢SQL百分比
    PostgreSQL 13: 系统视图pg_stat_activity新增leader_pid字段显示父进程信息
    PostgreSQL 13: 新增pg_stat_progress_analyze视图监控表分析进度
    PostgreSQL 13: pg_stat_statements视图新增执行计划耗时信息
    PostgreSQL 13: psql客户端提示符新增%x变量显示事务状态
    PostgreSQL 13: ALTER TABLE命令新增DROP EXPRESSION选项
    PostgreSQL 13: Reindexdb命令新增-j选项，支持全库并行索引重建
    PostgreSQL 13: 新增ignore_invalid_pages参数
    PostgreSQL 13: 普通表数据逻辑复制到分区表
    PostgreSQL 13: 多源逻辑复制实践
```

# 其它亮点
```
    PostgreSQL 13: 日期格式新增对FF1-FF6的支持
    PostgreSQL 13: 手册中新增术语(Glossary)附录
```

# 参考
[PostgreSQL 13 Beta 1 Released!](https://www.postgresql.org/about/news/postgresql-13-beta-1-released-2040/)
[E.1. Release 13](https://www.postgresql.org/docs/13/release-13.html)
[PostgreSQL 13 新特性汇总](https://postgres.fun/20200724165800.html)

