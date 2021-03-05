---
title: Zabbix编译安装 
date: 2020-08-11 11:16:08
tags: 
- zabbix
categories: 
- zabbix
top: 1
description: 
password: 

---
# 新建帐号

```
groupadd zabbix
useradd -g zabbix zabbix
```

<!-- more -->

# 安装依赖

```
sudo apt install libsnmp-dev
sudo apt install libssh2-1*
sudo apt install libopenipmi-dev
sudo apt install libldap2-dev
sudo apt install libevent-dev
sudo apt install curl
sudo apt install php-pgsql
sudo apt install php7.2-xml
sudo apt install apache2
sudo apt install php7.2-bcmath
sudo apt install php7.2-mbstring
sudo apt install php7.2-ldap
sudo apt install php7.0-snmp
```

# 下载  && 编译 && 安装

```
useradd -m zabbix -d /home/zabbix
su - zabbix
wget https://cdn.zabbix.com/zabbix/sources/stable/4.0/zabbix-4.0.2.tar.gz
tar -zxvf zabbix-4.0.2.tar.gz
cd zabbix-4.0.2/
./configure --prefix=/home/zabbix/release --enable-server --enable-proxy --enable-agent --enable-ipv6 --with-postgresql=/opt/pg11/bin/pg_config --with-net-snmp --with-ssh2 --with-openipmi --with-ldap --with-libcurl --with-iconv --enable-bcmath --enable-mbstring  --with-gd  --with-png-dir --with-jpeg-dir --with-freetype-dir
make && make install
```

# 配置zabbix

cd /home/zabbix/release/sbin
vim ../etc/zabbix_server.conf 填写正确DB内容
```
DBHost=DBHost
DBName=DBName
DBUser=DBUser
DBPassword='DBPassword'
```


# 启动服务

```
./zabbix_server
/etc/init.d/apache2 start
sudo mkdir /var/www/html/zabbix
sudo chown -R zabbix:zabbix
cp -fr /home/zabbix/zabbix-4.0.2/frontends/php/* /var/www/html/zabbix

```
页面就可以登陆了。

# 配置zabbix

```
http://hostname/zabbix/setup.php
```
![zabbix01](/images/zabbix01.png)


下一步，修改php配置保证所有状态直到OK

![zabbix02](/images/zabbix02.png)


# FAQ

PHP databases support off Fail
```
extension=/opt/remi/php72/root/usr/lib64/php/modules/pgsql.so
```
