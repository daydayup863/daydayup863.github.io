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

# 适用于PG的一个sysctl.conf配置
```
vim /etc/sysctl.conf

vm.dirty_background_bytes = 0
vm.dirty_background_ratio = 10
vm.dirty_bytes = 0
vm.dirty_expire_centisecs = 3000
vm.dirty_ratio = 20
vm.dirty_writeback_centisecs = 500

net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1    表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1  表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭；
net.ipv4.tcp_fin_timeout=30修改系統默认的 TIMEOUT 时间。

如果以上配置调优后性能还不理想，可继续修改一下配置：

vi /etc/sysctl.conf
net.ipv4.tcp_keepalive_time = 1200
#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
net.ipv4.ip_local_port_range = 1024 65000
#表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。
net.ipv4.tcp_max_syn_backlog = 8192
#表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_tw_buckets = 5000
#表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。
默认为180000，改为5000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，此项参数可以控制TIME_WAIT套接字的最大数量，避免Squid服务器被大量的TIME_WAIT套接字拖死。

```

# python 文件服务器
```
python2 -m SimpleHTTPServer 8080
python3 -m http.server
```

# 查看链接详情
```
jintao@jintao-ThinkPad-L490:~/personal/code/postgresql-master$ netstat -n | awk '/^tcp/ {++y[$NF]} END {for(w in y) print w, y[w]}'
CLOSE_WAIT 10
ESTABLISHED 153
TIME_WAIT 10

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
[postgres@l-grpxxxx ~]$ psql -Atc "select client_addr from pg_stat_activity where client_addr is not null" | sort|uniq -c| awk '{printf("%s %s ",$1,$2);cmd="host "$2; system(cmd)}'|awk '{printf("%10s %20s %s\n",$1,$2,$7)}'|sort -k 3
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
