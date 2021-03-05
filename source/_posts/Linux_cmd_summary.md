---
title: Linux 命令记录
date: 2020-08-19 11:15:42
tags: 
- Linux
- command
categories:
- Linux
top: 26
description: 
password: 

---

记录一些工作中遇到的命令.

<!-- more -->

# 磁盘读写测试
```
$ sudo hdparm -tT --direct /dev/nvme0n1p2

/dev/nvme0n1p2:
 Timing O_DIRECT cached reads:   2284 MB in  2.00 seconds = 1142.33 MB/sec
 Timing O_DIRECT disk reads: 100 MB in  0.08 seconds = 1226.93 MB/sec

# 顺序同步写入
$ time dd if=/dev/zero of=test bs=8k count=5120 oflag=dsync
5120+0 records in
5120+0 records out
41943040 bytes (42 MB, 40 MiB) copied, 4.98705 s, 8.4 MB/s

real	0m4.993s
user	0m0.019s
sys	0m0.872s

# 顺序写入 
$ time dd if=/dev/zero of=test bs=8k count=1000000
1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 10.7331 s, 763 MB/s

real	0m10.743s
user	0m0.189s
sys	0m6.423s

# 顺序读取
$ time dd if=test of=/dev/null bs=8k
1000000+0 records in
1000000+0 records out
8192000000 bytes (8.2 GB, 7.6 GiB) copied, 10.7331 s, 763 MB/s

real	0m10.743s
user	0m0.189s
sys	0m6.423s
```

# 启用/禁用CPU
```
$ sudo chcpu -d 3
CPU 3 disabled
$ sudo chcpu -e 3
CPU 3 enabled

```
# 增加/删除IP
```
# ip a a local 192.168.0.25/32 brd + dev bond0 && arping -q -c 3 -U -I  bond0 192.168.0.25

# ip a d local 192.168.0.25/32 brd + dev bond0

```
