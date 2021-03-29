---
title: Linux 新机配置 
date: 2021-03-29 15:32:31
tags: 
- linux
categories: 
- linux
top: 40
description: 
password: 

---

记录一些linux新机器常用的配置, 方便更换系统时更快的配置个人环境.

<!--more-->

# .bashrc相关配置

## 历史命令的上下翻页
```
bind '"\e[A":history-search-backward'
bind '"\e[B":history-search-forward'
```

## 历史命令保留个数，文件大小
```
HISTSIZE=100000
HISTFILESIZE=80000
```


