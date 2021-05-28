---
title: docker常用命令
date: 2021-03-11 16:57:05
tags: 
- docker
categories: 
- docker
top: 35
description: 
password: 

---


#将 image 文件从仓库抓取到本地， 默认library组
```
$ docker image pull library/hello-world
$ docker image pull hello-world

```

# 列出本机的所有 image 文件。
```
$ docker image ls
```

# 删除 image 文件
```
$ docker rmi [imageName]
```

<!--more-->

# 启动并进入终端
```
$ sudo docker run -i -t --name=docker_run centos /bin/bash
```

#进入docker终端$   
```
sudo docker run -it postgres:11.2  /bin/bash
```

#查看容器重启次数
```
docker inspect -f "{{.RestartCount }}" containerid
```

#查看容器最后一次的启动时间
```
docker inspect -f "{{.State.StartedAt }}" containerid
```

#获取container ip
```
docker inspect --format '{{.NetworkSettings.IPAddress}}'  containerid
```

#获取网络配置
```
docker incontainerid | jq '.[].NetworkSettings.Networks'
```
