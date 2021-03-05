---
title: PostgreSQL Postgis 源码安装
date: 2020-08-11 14:56:15
tags: 
- postgres
- postgis
- pg
- PostgreSQL
- postgresql
- geo
- gis
categories: 
- PostgreSQL
top: 15
description: 
password: 

---

为PostgreSQL 12.3 安装Postgis 2.5.4

# postgis 源码下载

巨慢, 耐心等待吧
```
postgres@jintao-ThinkPad-L490:~/download$ wget https://download.osgeo.org/postgis/source/postgis-2.5.4.tar.gz
--2020-08-11 14:57:46--  https://download.osgeo.org/postgis/source/postgis-2.5.4.tar.gz
Resolving download.osgeo.org (download.osgeo.org)... 140.211.15.30
Connecting to download.osgeo.org (download.osgeo.org)|140.211.15.30|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16041872 (15M) [application/octet-stream]
Saving to: ‘postgis-2.5.4.tar.gz’

postgis-2.5.4.tar.gz       6%[======>                                                    ]   1016K  5.72KB/s    eta 43m 7s
```
[其他版本下载地址](https://git.osgeo.org/gitea/postgis/postgis/branches/)
<!-- more -->

# 依赖下载 && 编译 && 安装

安装postgis2.5.4需要对应的包及其版本详见[install_requirements](https://postgis.net/docs/manual-2.5/postgis_installation.html#install_requirements)

## zlib1g-dev和libxml2
```
sudo apt-get install zlib1g-dev
sudo apt-get install libxml2-dev
```

## geos

编译时间很长
```
 wget http://download.osgeo.org/geos/geos-3.7.2.tar.bz2
 tar -jxvf geos-3.7.2.tar.bz2
 cd geos-3.7.2
 ./configure prefix=/home/postgres/pg12/geos
 make -j10
 make install
```

## proj

```
 wget  http://download.osgeo.org/proj/proj-4.9.2.tar.gz
 tar -zxvf proj-4.9.2.tar.gz
 cd proj-4.9.2/
 ./configure prefix=/home/postgres/pg12/proj
 make -j10
 make install
```

## gdal

编译时间很长
```
 wget http://download.osgeo.org/gdal/2.2.3/gdal-2.2.3.tar.gz
 tar -zxvf gdal-2.2.3.tar.gz
 cd gdal-2.2.3
 ./configure prefix=/home/postgres/pg12/gdal
 make -j10
 make install
```

## json-c

```
 wget  https://s3.amazonaws.com/json-c_releases/releases/json-c-0.15.tar.gz
 mkdir build
 cd build
 ../cmake-configure --prefix=/home/postgres/pg12/jsonc
 make -j10
 make install
```

## protobuf-c
不是必须, 需要使用ST_AsMVT则需要。
```
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.12.4/protobuf-all-3.12.4.tar.gz
tar -zxvf protobuf-all-3.12.4.tar.gz
cd protobuf-all-3.12.4
./configure
sudo make -j10
sudo make install
wget https://github.com/protobuf-c/protobuf-c/archive/v1.3.3.tar.gz
tar -zxvf v1.3.3.tar.gz
sudo vim /etc/ld.so.conf
/usr/local/protobuf-3.11.4/lib
/usr/local/protobuf-c-1.3.2/lib
export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
cd protobuf-c-1.3.3
./configure --prefix=/home/postgres/pg12/protobuf
make -j10
make install
```

# 更新动态库
sudo vim /etc/ld.so.conf 追加以下内容

```
/home/postgres/pg12/lib/
/home/postgres/pg12/jsonc/lib/
/home/postgres/pg12/geos/lib/
/home/postgres/pg12/proj/lib/
/home/postgres/pg12/gdal/lib/
/home/postgres/pg12/protobuf/lib/
```

# Postgis下载 && 编译 && 安装
```
 wget [http://download.osgeo.org/postgis/source/postgis-2.5.4.tar.gz
 tar -zxvf postgis-2.5.4.tar.gz
 cd postgis-2.5.4
 ./configure --with-pgconfig=/home/postgres/pg12/bin/pg_config --with-xml2config=/usr/bin/xml2-config --with-geosconfig=/home/postgres/pg12/geos/bin/geos-config --with-gdalconfig=/home/postgres/pg12/gdal/bin/gdal-config --with-projdir=/home/postgres/pg12/proj  --with-jsondir=/home/postgres/pg12/jsonc --with-protobufdir=/home/postgres/pg12/protobuf --with-gui --with-topology --with-raster
 make -j10 
 make install 
```

# 创建extension

```
postgres@jintao-ThinkPad-L490:~/download/postgis-2.5.4$ psql
psql (12.3)
Type "help" for help.

postgres=# create extension postgis;
CREATE EXTENSION
postgres=# create extension postgis_t;
postgis_tiger_geocoder  postgis_topology
postgres=# create extension postgis_tiger_geocoder;
ERROR:  required extension "fuzzystrmatch" is not installed
HINT:  Use CREATE EXTENSION ... CASCADE to install required extensions too.
postgres=# create extension postgis_tiger_geocoder cascade;
NOTICE:  installing required extension "fuzzystrmatch"
CREATE EXTENSION
postgres=# create extension postgis_topology;
postgis_topology
postgres=# create extension postgis_topology;
CREATE EXTENSION
postgres=# \dx
                                            List of installed extensions
          Name          | Version |   Schema   |                             Description
------------------------+---------+------------+---------------------------------------------------------------------
 fuzzystrmatch          | 1.1     | public     | determine similarities and distance between strings
 plpgsql                | 1.0     | pg_catalog | PL/pgSQL procedural language
 postgis                | 2.5.4   | public     | PostGIS geometry, geography, and raster spatial types and functions
 postgis_tiger_geocoder | 2.5.4   | tiger      | PostGIS tiger geocoder and reverse geocoder
 postgis_topology       | 2.5.4   | topology   | PostGIS topology spatial types and functions
(5 rows)

postgres=# select postgis_full_version();
                                                                                                   postgis_full_version

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
-------
 POSTGIS="2.5.4" [EXTENSION] PGSQL="120" GEOS="3.7.2-CAPI-1.11.2 b55d2125" PROJ="Rel. 4.9.2, 08 September 2015" GDAL="GDAL 2.2.3, released 2017/11/20" LIBXML="2.9.10" LIBJSON="0.15" LIBPROTOBUF="1.3.3" TOPOLOGY
RASTER
(1 row)

postgres=#
postgres=# select st_geographyfromtext('SRID=4326;POINT(121 23)');
                st_geographyfromtext
----------------------------------------------------
 0101000020E61000000000000000405E400000000000003740
(1 row)

postgres=#
```
**Note:**
最新的CREATE EXTENSION 详见 http://postgis.net/install/
