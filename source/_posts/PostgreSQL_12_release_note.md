---
title: PostgreSQL 12 新特性汇总
date: 2021-04-30 11:08:32
tags: 
- PostgreSQL
- release note
categories: 
- PostgreSQL
- release note
top: 48
description: 
password: 

---


尽管现阶段 PostgreSQL 12 才出 Beta3 版本，但 12 版本的新特性和正式版不会有太大出入，最近抽时间对 12 版本的新特性进行了探索，整体上 12 版本的变化不小。

12 版本的典型新特性如下:

    支持 SQL/JSON path
    支持 Generated Columns
    CTE 支持 Inlined With Queries
    新增 Pluggable Table Storage Interface
    分区表性能大辐提升
    在线重建索引(Reindex Concurrently)

<!--more-->

详见以下，文中的链接对每一个特性进行了介绍。
新功能

12 版本新功能主要包括 JSON path queries 、Generated Columns、Pluggable Table Storage Interface，如下:

    PostgreSQL 12: 支持 SQL/JSON path 特性
    PostgreSQL 12: 支持 Generated Columns 特性
    PostgreSQL 12: 新增 Pluggable Table Storage Interface

性能优化

12 版本性能提升主要体现在分区表性能提升、CTE 支持 Inlined With Queries、Btree 索引性能提升等，如下:

    PostgreSQL 12: 分区表DML性能大辐提升
    PostgreSQL 12: 分区表数据导入性能提升
    PostgreSQL 12: CTE 支持 Inlined With Queries
    PostgreSQL 12: 新增 plan_cache_mode 参数设置执行计划策略

备份复制相关

备份、复制相关变化较大，包括配置文件的变化、新增流复制备库激活方式、max_wal_senders连接数变化等，如下:

    PostgreSQL 12: Recovery.conf 文件参数合并到 postgresql.conf
    PostgreSQL 12: 新增 pg_promote() 函数用于激活备库(流复制主备切换)
    PostgreSQL 12: COPY FROM 命令支持 WHERE 过滤条件
    PostgreSQL 12: max_wal_senders 连接数从 max_connections 剥离

监控相关

监控方面主要体现在支持在线重建索引、新增 pg_stat_progress_create_index 视图监控索引创建进度、新增 log_statement_sample_rate 参数控制数据库日志中慢SQL百分比等，如下:

    PostgreSQL 12: 支持在线重建索引(Reindex Concurrently)
    PostgreSQL 12: 新增 pg_stat_progress_create_index 视图监控索引创建进度
    PostgreSQL 12: 新增 log_statement_sample_rate 参数控制数据库日志中慢SQL百分比
    PostgreSQL 12: 新增 pg_partition_tree() 函数显示分区表信息

其它

其它方面的增强主要体现在命令行工具，如下:

    PostgreSQL 12: psql 命令增强
    PostgreSQL 12: EXPLAIN 新增 SETTINGS 选项显示非默认优化器参数
    PostgreSQL 12: pgbench 新增 \gset 命令支持将SQL结果存入变量
    PostgreSQL 12: VACUUM 新增 INDEX_CLEANUP 选项控制是否回收索引

参考

[PostgreSQL 12 Beta 1 Released!](https://www.postgresql.org/about/news/postgresql-12-beta-1-released-1943/)


