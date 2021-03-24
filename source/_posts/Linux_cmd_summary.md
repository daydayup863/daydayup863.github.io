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

# python 文件服务器
```
python2 -m SimpleHTTPServer 8080
python3 -m http.server
```


# curl 中带有中文时, 需要使用--data-urlencode否则会报400
```
 curl -G "http://192.168.0.1/api/postgresql/get_platform_info" --data-urlencode "bu=公共部门"
```


# 查找ip 对应的条数以及主机名
```
########   pgbouncer
[postgres@l-pgb1zzzz ~]$ psql -Upostgres -d pgbouncer -Atc "show clients"|awk -F "|" '{print $5}'|grep "^1"|sort|uniq -c|sort -nk 1|awk '{printf("%s %s ", $1, $2); cmd="host "$2; system(cmd)}'|awk '{printf("%5s %15s %s \n", $1,$2,$7)}'
    1 192.86.238.131 host1
    1 192.88.105.131 host2
    8 192168.254.191 host1
[postgres@l-pgb1zzzz ~]$


####### postgresql
[postgres@l-grpendb2.cc.cna ~]$ psql -Atc "select client_addr from pg_stat_activity where client_addr is not null" | sort|uniq -c| awk '{printf("%s %s ",$1,$2);cmd="host "$2; system(cmd)}'|awk '{printf("%10s %20s %s\n",$1,$2,$7)}'|sort -k 3
         1         19288.101.77 l-callcenter14.h.cn6
         1         19286.236.29 l-host14.xxxx
         1         19290.4.23   l-host1.yyyy
         1         19290.15.110 l-host7.yyyy
         1         19290.15.111 l-host8.yyyy
```

# PostgreSQL日志变成一行
```
awk '{if ($0 !~  "^[\t]") printf("\n%s" ,$0); else printf $0}' postgresql-Tue.log
zcat postgresql-2013-02-2*.log.gz | perl -e 'while(<>){ if($_ !~ /^\t/ig) { chomp; print "\n",$_;} else {chomp; print;}}' |less
```

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
