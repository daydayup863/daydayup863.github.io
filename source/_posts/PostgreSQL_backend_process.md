---
title: PostgreSQL后端进程 
date: 2020-08-27 17:32:48
tags: 
- PostgreSQL
- backend
- Architecture
categories: 
- PostgreSQL
top: 20
description: 
password: 

---

# 整体架构图

![ An example of the process architecture in PostgreSQL ](/images/A_example_of_the_process_architecture_in_PostgreSQL.png)

<!-- more -->

# checkpoint
## 功能描述

其保证了数据库的一致性状态，定期执行检查点是很重要的，确保数据变化持久保存到磁盘中并且数据库的状态是一致的。
## 参数

checkpoint_timeout
系统自动执行checkpoint之间的最大时间间隔。系统默认值是5分钟，这个值可以在压测过程中调大，尽量避免执行checkpoint争抢IO

max_wal_size
写满多少个WAL时执行checkpoint，也是同理，这个值可以在压测过程中调大，尽量避免执行checkpoint争抢IO

min_wal_size
只要wal日志目录使用空间小于该值，那么旧的wal日志就会循环使用而不是进行删除。这个参数是为了确保足够的wal空间预留给突发情况，比如大的跑批操作。

checkpoint_completion_target
分散检查点，默认为0.5，即表示每个checkpoint需要在checkpoints间隔时间的50%内完成，然后立马进行fsync，fsync执行是很快的(为了平滑fsync，以防尖锐的IO请求，PostgreSQL9.6以后加了checkpoint_flush_after、wal_writer_flush_after、bgwriter_flush_after和backend_flush_after这些参数来缓解)看如下一个场景。
| 场景                             | 数据量 | 数据写入速度( 1gb/ s )          |
| -                                | -      | -                               |
| checkpoint_completion_target=0.5 | 100G   | 100/ ( 0.53060 ) 1024 ≈ 114 M/s |
| checkpoint_timeout = 30min       | 100G   | 100/ ( 0.8 3060 ) 1024 ≈ 71 M/s |

full_page_writes
PostgreSQL服务器在检查点之后对页面的第一次写入时将整个页面写到WAL里面。如果checkpoint发生太频繁，会导致写放大，默认为on，假如调为off，需要确保数据库在压测期间不要崩溃，不然重启后可能发生数据块部分写，导致重启失败。full_page_writes就是为了确保数据页一致性，不发生块折断。而在MySQL中，则是通过double write来预防partial write的。如果你的块设备对齐，并支持原子写(原子写大于或等于一个DATA FILE数据页的大小)，那可以关闭这个参数

# background writer
## 功能描述

1、数据库在进行查询处理时若发现要读取的数据不在缓冲区中时要先从磁盘中读入需要的页面，此时如果缓冲区已满，则需要遵从类似于LRU算法先选择部分缓冲区中的页面置换出去。如果被替换的页面没有被修改过，则可以直接丢弃；但如果已经被修改过，则需要先将这些页面写出到磁盘后才能置换，通过使用BgWriter定期写出缓冲区中的部分脏页，为缓冲区腾出空间，就可以降低查询处理被阻塞的可能性；

2、PostgreSQL在定期做检查点时需要把所有脏页写出到磁盘，通过BgWriter预先写出一些脏页，可以减少检查点时要进行的IO动作，使系统的IO更加平稳。通过BgWriter对共享缓冲区写操作的管理，避免了其他服务进程在需要读入新的页面到缓冲区时，不得不先进行写盘的操作。

## 参数

writer_delay
background writer每次扫描之间的时间间隔，也就是刷shared buffer脏页的进程调度间隔，尽量高频调度，减少用户进程申请不到内存而需要主动刷脏页的可能(导致RT升高)

bgwriter_lru_maxpages
一次最多刷多少脏页

bgwriter_lru_multiplier
写出至多bgwriter_lru_multiplier * N个脏页，并且不超过bgwriter_lru_maxpages值的限制。其中N是最近一段时间在两次BgWriter运行期间系统新申请的缓冲区页数。后台写进程根据最近服务进程需要的buffer数量乘上这个比率估算出下次服务进程需要的buffer数量，再使用后台写进程刷脏页面，使缓冲区能使用的干净页面达到这个估计值

bgwriter_flush_after
每当bgwriter写入的字节数超过bgwriter_flush_after时，就会强制OS从page cache中写出。这样做将限制page cache中脏数据量，从而减少在检查点末尾发出fsync或操作系统在后台大批量写回数据时出现停顿的可能性
