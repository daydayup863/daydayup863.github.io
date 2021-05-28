---
title: CentOS7+PostgreSQL11.2+Docker
date: 2021-03-11 16:36:07
tags:
- docker
- PostgreSQL
categories: 
- PostgreSQL
- docker
top: 34
description: PostgreSQL Docker image 制作
password: 

---

# OS image
## 编写centos7.sh
```
#!/bin/bash
mkdir centos7
cd centos7
supermin5 -v --prepare bash grep yum yum-plugin rpm systemd pkgconfig initscripts -o supermin.d
supermin5 -v --build --format chroot supermin.d -o appliance.d
cp /etc/resolv.conf appliance.d/etc/
echo 7 > appliance.d/etc/yum/vars/releasever
tar --numeric-owner -cpf centos-7.tar -C appliance.d .
cat centos-7.tar | docker import - os/centos
```

<!--more-->

## 生成OS image
```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
os/centos           latest              88805d994ae8        About a minute ago   279MB
```

# PostgreSQL image

## 初始化PostgreSQL基础信息脚本
```
#!/bin/bash
su postgres <<EOF
/opt/pg11/bin/pg_ctl start -D /export/pg110_data;
exit;
EOF
```
## 自定义dockerfile
```
FROM os/centos
 
RUN localedef -c -f UTF-8 -i en_US en_US.utf8
 
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:zh \
    LC_ALL=en_US.UTF-8
 
MAINTAINER taot.jin <84660320@qq.com>
 
RUN mkdir /export
ADD ./pg110_data /export/pg110_data
 
RUN mkdir /export/rpms
ADD ./rpms/*.rpm /export/rpms/
#RUN rpm -ivh  /export/rpms/geos-*.rpm  /export/rpms/proj-*.rpm /export/rpms/gdal-*.rpm /export/rpms/json-c*.rpm  /export/rpms/postgis-*.rpm
 
RUN useradd postgres
RUN chown -R postgres:postgres /export/pg110_data
RUN chmod 0700 /export/pg110_data
 
RUN rm /etc/yum.repos.d/*.repo
ADD ./my.repo /etc/yum.repos.d/
 
ENV LD_LIBRARY_PATH "/opt/pg11/lib:/usr/lib64:/lib64:/usr/lib:/lib"
ENV PATH  "/opt/pg11/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
 
RUN ldconfig
 
RUN yum clean all && yum makecache && yum install -y postgresql wget pkgconfig
 
ADD ./start_postgres.sh /start_postgres.sh
RUN chmod +x /start_postgres.sh
 
#RUN echo "su postgres -c \"/opt/pg11/bin/pg_ctl -D /export/pg110_data restart &>/export/pg110_data/log/postgre_start.log\"" >> /etc/rc.local
 
VOLUME ["/export/pg110_data"]
VOLUME ["/opt/pg11"]
EXPOSE 5432
 
#RUN echo "host    all             all             0.0.0.0/0               md5" >> /var/lib/pgsql/data/pg_hba.conf
 
CMD ["/start_postgres.sh"]
```

## 生成image
```
# docker image build -t postgresql11.2.1 .
```

# 数据持久化
```
docker volume ls
docker volume create pg110_data
docker volume create opt_pg11_1


docker run -it -v pg110_data:/export/pg110_data -v  opt_pg11_1:/opt/pg11 postgresql:11.2.1
docker run -it --mount type=bind,source=/export,target=/export postgresql:11.2.1
```

# 更新image
```
# rpm -qa|grep  proj
# rpm -ivh proj-4.9.2-1.el7.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:proj-4.9.2-1.el7                 ################################# [100%]
# rpm -qa|grep proj
proj-4.9.2-1.el7.x86_64
# docker container ps -l
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                       PORTS               NAMES
5aeefd7f48e9        postgresql:11.2.1   "/bin/sh -c /start_p…"   About a minute ago   Exited (130) 9 seconds ago                       zealous_ganguly

# docker commit -m "test commit" 5aeefd7f48e9 postgresql:11.2.2
sha256:df7a3a3cff7fd017e889dce7d03885e65f1b512ad3e443ab8254e5375805e24d

# docker run -it -v pg110_data:/export/pg110_data -v  opt_pg11_1:/opt/pg11 postgresql:11.2.2
pg_ctl: another server might be running; trying to start server anyway
waiting for server to start....[    2019-03-04 17:23:41.736 CST 10 5c7cee9d.a 1  0]LOG:  listening on IPv4 address "0.0.0.0", port 5432
[    2019-03-04 17:23:41.736 CST 10 5c7cee9d.a 2  0]LOG:  listening on IPv6 address "::", port 5432
[    2019-03-04 17:23:41.740 CST 10 5c7cee9d.a 3  0]LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
[    2019-03-04 17:23:41.766 CST 10 5c7cee9d.a 4  0]LOG:  redirecting log output to logging collector process
[    2019-03-04 17:23:41.766 CST 10 5c7cee9d.a 5  0]HINT:  Future log output will appear in directory "log".
 done
server started
```


# image迁移
## save/load
### save导出
```
docker save c92217432f48 > postgresql.tar
```

### load导入
```
docker load < postgresql.tar
docker tag c92217432f48 postgresql:11.2.4
```

## export/import
### export导出
```
# docker container ps -l
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
9caea184f25b        postgresql:11.2.1   "/bin/sh -c /start_p…"   6 minutes ago       Exited (0) 8 seconds ago                       relaxed_benz

# docker export 9caea184f25b > postgresql.tar
```

### import导入
```
cat postgresql.tar |docker import - postgresql:11.2.2
```


# 错误处理
```
https://www.libaocai.com/2767.html
```
