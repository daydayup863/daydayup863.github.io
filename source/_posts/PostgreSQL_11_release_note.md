---
title: PostgreSQL 11 新特性汇总
date: 2021-04-30 11:08:32
tags: 
- PostgreSQL
- release note
categories: 
- PostgreSQL
- release note
top: 47 
description: 
password: 

---


2018-10-18 PostgreSQL官网 宣布 PostgreSQL 11 正式版发行，PostgreSQL 11 重点对性能进行了提升和功能完善，特别是对大数据库和高计算负载的情况下进行了增强，主要包括以下:

    对分区表进行了大幅的改进和增强。
    增加了对存储过程的支持，存储过程支持事务。
    增强了并行查询能力和并行数据定义能力。
    增加了对 just-in-time (JIT) 编译的支持，加速SQL中的表达式执行效率。

最近对PostgreSQL以上新特性和其它功能完善做了演示，希望对PostgreSQL从业者有帮助，详见以下。
分区表的改进

<!--more-->

PostgreSQL 11 对分区表进行了重大的改进，例如增加了哈希分区、支持创建主键、外键、索引、支持UPDATE分区键以及增加了默认分区，这些功能的完善极大的增强了分区表的可用性，详见以下:

    PostgreSQL11: 分区表增加哈希分区
    PostgreSQL11：分区表支持创建主键、外键、索引
    PostgreSQL11: 分区表支持UPDATE分区键
    PostgreSQL11: 分区表增加 Default Partition

支持存储过程

PostgreSQL 11 版本一个重量级新特性是对存储过程的支持，同时支持存储过程嵌入事务，存储过程是很多 PostgreSQL 从业者期待已久的特性，尤其是很多从Oracle转到PostgreSQL朋友。

尽管PostgreSQL提供函数可以实现大多数存储过程的功能，但函数不支持部分提交，换句话说，函数中的SQL要么都执行成功，要不全部返回失败，详见以下:

    PostgreSQL11: 支持存储过程(SQL Stored Procedures)

并行能力的增强

PostgreSQL 11 版本在并行方面得到较大增强，例如支持并行创建索引、并行Hash Join、并行 CREATE TABLE .. AS等，详见以下:

    PostgreSQL11：支持并行创建索引(Parallel Index Builds)
    PostgreSQL11：支持并行哈希连接(Parallel Hash Joins)

增加对Just-in-Time (JIT)编译的支持

PostgreSQL 11 版本的一个重量级新特性是引入了 JIT (Just-in-Time) 编译来加速SQL中的表达式计算效率。

JIT 表达式的编译使用LLVM项目编译器来提升在WHERE条件、指定列表、聚合以及一些内部操作表达式的编译执行，详见以下:

    PostgreSQL11: 增加对JIT(just-in-time)编译的支持提升分析型SQL执行效率

其它功能完善

此外， PostgreSQL 11 增强了其它新特性以增加用户体验，以下列举了主要的几点，详见以下:

    PostgreSQL11: 新增非空默认值字段不需要重写表
    PostgreSQL11: Indexs With Include Columns
    PostgreSQL11: 新增三个默认角色
    PostgreSQL11: 可通过GRNAT权限下放的四个系统函数
    PostgreSQL11: Initdb/Pg_resetwal支持修改WAL文件大小
    PostgreSQL11: psql 新增 \gdesc 显示查询结果的列名和类型
    PostgreSQL11: psql 新增变量记录SQL语句的执行情况和错误

关于PostgreSQL

PostgreSQL 号称世界上最先进的开源关系型数据库，PostgreSQL 全球社区是一个由数千名用户、开发人员、公司或其他组织组成。 PostgreSQL 起源于加利福利亚的伯克利大学，有30年以上历史，经历了无数次开发升级。

PostgreSQL 的出众之处在于不仅具有商业数据库的功能特性，同时在扩展性、安全性、稳定性等高级数据库特性方面超越了它们。

若想获取到更多关于PostgreSQL的信息或者加入PostgreSQL社区，请浏览官网 PostgreSQL.org 。

参考

    [PostgreSQL 11 Released!](https://www.postgresql.org/about/news/postgresql-11-released-1894/)
    [PostgreSQL 11 有哪些引人瞩目的新特性？](https://postgres.fun/20181102084300.html)


