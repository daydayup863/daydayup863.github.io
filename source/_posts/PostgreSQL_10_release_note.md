---
title: PostgreSQL 10 新特性汇总
date: 2021-04-30 11:08:32
tags: 
- PostgreSQL
- release note
categories: 
- PostgreSQL
- release note
top: 46 
description: 
password: 

---

PostgreSQL10Beta1 版本于 2017年5月18日发行，PostgreSQL 10 新增了大量新特性，其中特重量级新特性如下：

    内置分区表（ Native Table Partitioning）
    逻辑复制（Logical Replication）
    并行功能增强（Enhancement of Parallel Query）
    Quorum Commit for Synchronous Replication
    全文检索支持JSON和JSONB数据类型

其它新特性详见 PostgreSQL10 Release ，这里不详细列出，由于时间和精力的关系，目前仅对部分新特性进行演示，详见以下博客：

    PostgreSQL10：重量级新特性-支持分区表
    PostgreSQL10：Parallel Queries 增强
    PostgreSQL10：Additional FDW Push-Down
    PostgreSQL10：逻辑复制（Logical Replication）之一
    PostgreSQL10：逻辑复制（Logical Replication）之二
    PostgreSQL10：Quorum Commit for Synchronous Replication
    PostgreSQL10：Multi-column Correlation Statistics
    PostgreSQL10：新增 pg_hba_file_rules 视图
    PostgreSQL10：全文检索支持 JSON 和 JSONB
    PostgreSQL10：Identity Columns 特性介绍
    PostgreSQL10：Incompatible Changes
    PostgreSQL10：新增 pg_sequence 系统表


参考
[E.17. Release 10](https://www.postgresql.org/docs/10/release-10.html)
