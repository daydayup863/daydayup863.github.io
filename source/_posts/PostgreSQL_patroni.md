---
title: PostgreSQL HA patroni 测试案例 
date: 2021-03-16 18:28:12
tags: 
categories: 
top: 
description: 
password: 

---
主要针对patroni 由1.6.5-2 升级至2.0.1 的版本

#测试环境
    zookeeper + CentOS 7 + pg11

# Switchover (计划内维护)
## 主从切换
            case 1. 期望原主库降为新从库的场景
            case 2. 期望原主库停掉不做后续处理场景
## DB集群切换下线1个HA管理的从节点
## DB集群剔除1个HA管理的从节点
## 新加从节点
            case 1. DB集群新加从节点, 期望不需要加入HA管理的场景
            case 2. DB集群新加从节点, 期望需要加入HA管理的场景
# failover (故障切换)
##  Master
### 主库DB实例异常kill, 导致主库down
### 主库DB实例通过pg_ctl 停掉, 导致主库down
### 主库DB实例 patroni down (kill)
### 主库DB实例 patroni down (kill -9)
### 主库DB实例所在DB Server  down
### 主库性能抖动的影响
### 主库网络抖动的影响
### 主从复制断掉影响
### base目录被删除
## Slave
### 从库DB实例异常kill, 导致从库down
### 从库DB实例通过pg_ctl 停掉, 导致从库down
### 从库DB实例 patroni down (kill )
### 从库DB实例 patroni down (kill -9)
### 从库DB实例所在DB Server  down
### 从库性能抖动的影响
### 从库网络抖动的影响
# zk故障
## zk仅到DB集群主库网络不通, zk至DB集群至少有个1个从库都是通的
## zk到DB集群的所有从库网络都不通, zk至DB集群主库是通的
## zk到DB集群的所有节点网络都几乎同时不通,包括主从库
### 几乎同时不通, 又几乎同时通
### 几乎同时不通, 主库先通, 从库后通
### 几乎同时不通, 从库先通, 过1个ttl, 主库再通
### 几乎同时不通, 从库先通, 但不到1个ttl, 主库就通了
## zk服务down掉, zk到DB集群的所有节点网络正常, 但是都无法正常访问zk
