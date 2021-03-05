---
title: 内核Softdog测试
date: 2020-08-05
tags: [ Linux, softdog, watchdog ]
categories: Linux
top: 10
description: Linux内核watchdog于监视系统是否正在运行。由于存在不可恢复的软件错误，应该自动重新启动挂起的系统
password: 

---

# Watchdog的功能
    Linux内核watchdog于监视系统是否正在运行。由于存在不可恢复的软件错误，应该自动重新启动挂起的系统。

    看门狗模块特依赖所使用的硬件或芯片。个人计算机用户通常不需要看门狗，因为他们可以手动重置系统。但是，watchdog对于任务关键型系统和需要无需人工干预即可自行重启的系统很有用。
例如，需要自动硬件重置功能的远程位置的服务器。

<!-- more -->



# Watchdog的实现
    看门狗功能设置了一个计时器在预定时间后超时。然后看门狗软件定期喂狗(刷新硬件计时器)。如果停止喂狗，到达设定时间后，将会出现狗咬人(执行设备的硬件重置)。
可以分为hardware watchdog 和 software watchdog。
    hardware watchdog 是主板芯片的功能。不同的芯片使用不同的模块，例如：Intel 主板使用 “iTCO_wdt”， HP 通常使用 “hpwdt”，IBM 使用 “vmwatchdog”，Xen VM 使用 “xen_wdt”。
    software watchdog 是Linux内核提供使用内核计时器实现的软件监视程序。

# 测试Linux自带watchdog
```
[root@server6 ~]# ls /dev/watchdog*              #查看watchdog设备文件
/dev/watchdog  /dev/watchdog0
[root@server6 ~]# lsmod |grep -i soft            #查看softdog是否加载
[root@server6 ~]# modinfo softdog                #查看softdog模块信息
filename:       /lib/modules/3.10.0-957.10.1.el7.x86_64/kernel/drivers/watchdog/softdog.ko.xz
alias:          char-major-10-130
license:        GPL
description:    Software Watchdog Device Driver
author:         Alan Cox
retpoline:      Y
rhelversion:    7.6
srcversion:     250C077ED39E75BF25AD8D1
depends:
intree:         Y
vermagic:       3.10.0-957.10.1.el7.x86_64 SMP mod_unload modversions
signer:         CentOS Linux kernel signing key
sig_key:        17:EA:5F:B9:16:4B:C2:26:55:5C:00:43:FA:D4:E5:86:CC:E8:A2:05
sig_hashalgo:   sha256
parm:           soft_margin:Watchdog soft_margin in seconds. (0 < soft_margin < 65536, default=60) (uint)
parm:           nowayout:Watchdog cannot be stopped once started (default=0) (bool)
parm:           soft_noboot:Softdog action, set to 1 to ignore reboots, 0 to reboot (default=0) (int)
parm:           soft_panic:Softdog action, set to 1 to panic, 0 to reboot (default=0) (int)

[root@server6 ~]# modprobe softdog               #加载softdog 可以设置 soft_margin soft_noboot   soft_panic   nowayout等参数
[root@server6 ~]# ls /dev/watchdog*              #查看watchdog设备文件
/dev/watchdog  /dev/watchdog0  /dev/watchdog1
[root@server6 ~]# wdctl /dev/watchdog1           #查看softdog设备文件信息
Device:        /dev/watchdog1
Identity:      Software Watchdog [version 0]
Timeout:       60 seconds
Pre-timeout:    0 seconds
FLAG           DESCRIPTION               STATUS BOOT-STATUS
KEEPALIVEPING  Keep alive ping reply          1           0
MAGICCLOSE     Supports magic close char      0           0
SETTIMEOUT     Set timeout (in seconds)       0           0
[root@server6 ~]# ehco a >/dev/watchdog          #追加除V以外任意字符到softdog  60秒后系统将重启
watchdog   watchdog0  watchdog1
[root@server6 ~]# echo V >/dev/watchdog1         #取消测试
```
# 警告

```
错误的watchdog配置可能导致以下情形
    无限重启
    硬重置导致文件损坏
    不可预测的重启
线上服务器慎用。
```
#  参考
```
    https://linuxhint.com/linux-kernel-watchdog-explained/
```
