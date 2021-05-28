---
title: PostgreSQL WAL 记录了什么?
date: 2021-03-15 14:37:02
tags:
- WAL
- PostgreSQL
categories: 
- WAL
- PostgreSQL
top: 38
description: 
password: 

---



# 关于持久性
```
持久性是指，事务提交后，对系统的影响必须是永久的，即使系统意外宕机，也必须确保事务提交时的修改已真正永久写入到永久存储中。
最简单的实现方法，当然是在事务提交后立即刷新事务修改后的数据到磁盘。但是磁盘和内存之间的IO操作是最影响数据库系统影响时间的，一有事务提交就去刷新磁盘，会对数据库性能产生不好影响。
```

<!--more-->

# WAL记录了什么
XLOG Record按存储的数据内容来划分，大体可以分为三类
## Record for backup block
```
    存储full-write-page的block，这种类型Record是为了解决page部分写的问题。在checkpoint完成后第一次修改数据page，在记录此变更写入事务日志文件时整页写入（需设置相应的初始化参数，默认为打开）；
```
## Record for tuple data block：
```
    存储page中的tuple变更，使用这种类型的Record记录；
```
## Record for Checkpoint
```
    在checkpoint发生时，在事务日志文件中记录checkpoint信息(其中包括Redo point)。
```

# WAL机制的引入
```
WAL机制的引入，即保证了事务持久性和数据完整性，又尽量地避免了频繁IO对性能的影响。
```


# WAL过程分析

```
先写到缓冲区Buffer-再刷新到磁盘Disk。
WAL机制实际是在这个写数据的过程中加入了对应的写wal log的过程，步骤一样是先到Buffer，再刷新到Disk。

Change发生时：先将变更后内容记入WAL Buffer,再将更新后的数据写入Data Buffer
Commit发生时：WAL Buffer刷新到Disk, Data Buffer写磁盘推迟
Checkpoint发生时：将所有Data Buffer刷新到磁盘
数据发生变动时: commit和checkpoint

```

# WAL的好处

```
通过上面的分析，可以看到：

当宕机发生时，Data Buffer的内容还没有全部写入到永久存储中，数据丢失；但是WAL Buffer的内容已写入磁盘，根据WAL日志的内容，可以恢复库丢失的内容。
在提交时，仅把WAL刷新到了磁盘，而不是Data刷新：
```

从IO次数来说，WAL刷新是少量IO，Data刷新是大量IO，WAL刷新次数少得多；
从IO花销来说，WAL刷新是连续IO，Data刷新是随机IO，WAL刷新花销小得多。
因此WAL机制在保证事务持久性和数据完整性的同时，成功地提升了系统性能。

