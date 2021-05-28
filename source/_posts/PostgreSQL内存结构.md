---
title: PostgreSQL内存结构 
date: 2021-05-28 14:10:41
tags: 
- PostgreSQL
- memory architecture
categories: 
- PostgreSQL
- memory architecture
top: 53 
description: 
password: postgresql

---


在PostgreSQL内存结构中，主要分为本地缓冲区(Local Memory Area)和共享缓冲区(hared Memory Area)两部分, 本文主要介绍这两种缓冲区的功能。

<!--more-->

# 本地缓冲区

当连接建立时, postgres都会为每一个backend process分配Local Memory Area，每一块区域又分为三个子区域, 如下表所示:



Each backend process allocates a local memory area for query processing; each area is divided into several sub-areas – whose sizes are either fixed or variable



# 共享缓冲区









