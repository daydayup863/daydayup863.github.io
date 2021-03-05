---
title: PostGIS 操作geometry方法
date: 2020-12-24 15:42:57
tags: 
- gis
- postgis
- PostgreSQL
categories: 
- PostgreSQL
- gis
- Postgis
top: 26
description: 
password: 

---

记录Postgis常用方法.

<!-- more -->

# WKT定义几何对象格式：
```
POINT(0 0) ——点
LINESTRING(0 0,1 1,1 2) ——线
POLYGON((0 0,4 0,4 4,0 4,0 0),(1 1, 2 1, 2 2, 1 2,1 1)) ——面
MULTIPOINT(0 0,1 2) ——多点
MULTILINESTRING((0 0,1 1,1 2),(2 3,3 2,5 4)) ——多线
MULTIPOLYGON(((0 0,4 0,4 4,0 4,0 0),(1 1,2 1,2 2,1 2,1 1)), ((-1 -1,-1 -2,-2 -2,-2 -1,-1 -1))) ——多面
GEOMETRYCOLLECTION(POINT(2 3),LINESTRING((2 3,3 4))) ——几何集合
```

# 常用函数：

```
wkt转geometry st_geomfromtext(wkt,wkid)
geometry转wkt st_astext(geom)
获取点对象x、y坐标值 st_x(geom)、st_y(geom)
获取线/面对象四至 st_xmin(geom)、st_ymin(geom)、st_xmax(geom)、st_ymax(geom)
计算两点之间距离 st_distance(geom,geom) / st_distance(wkt,wkt)
计算线的长度 st_length(geom) / st_length(wkt)
计算面积 st_area(geom) / st_area(wkt)
缓冲区计算 st_buffer(geom,distance) / st_buffer(wkt,distance)

```


# 管理函数

```
添加几何字段 AddGeometryColumn(, , , , , )
删除几何字段 DropGeometryColumn(, , )
检查数据库几何字段并在geometry_columns中归档 Probe_Geometry_Columns()
给几何对象设置空间参考（在通过一个范围做空间查询时常用） ST_SetSRID(geometry, integer)
```

# 几何对象关系函数
```
获取两个几何对象间的距离 ST_Distance(geometry, geometry)
如果两个几何对象间距离在给定值范围内，则返回TRUE ST_DWithin(geometry, geometry, float)
判断两个几何对象是否相等
（比如LINESTRING(0 0, 2 2)和LINESTRING(0 0, 1 1, 2 2)是相同的几何对象） ST_Equals(geometry, geometry)
判断两个几何对象是否分离 ST_Disjoint(geometry, geometry)
判断两个几何对象是否相交 ST_Intersects(geometry, geometry)
判断两个几何对象的边缘是否接触 ST_Touches(geometry, geometry)
判断两个几何对象是否互相穿过 ST_Crosses(geometry, geometry)
判断A是否被B包含 ST_Within(geometry A, geometry B)
判断两个几何对象是否是重叠 ST_Overlaps(geometry, geometry)
判断A是否包含B ST_Contains(geometry A, geometry B)
判断A是否覆盖 B ST_Covers(geometry A, geometry B)
判断A是否被B所覆盖 ST_CoveredBy(geometry A, geometry B)
通过DE-9IM 矩阵判断两个几何对象的关系是否成立 ST_Relate(geometry, geometry, intersectionPatternMatrix)
获得两个几何对象的关系（DE-9IM矩阵） ST_Relate(geometry, geometry)
```

#几何对象处理函数：
```
获取几何对象的中心 ST_Centroid(geometry)
面积量测 ST_Area(geometry)
长度量测 ST_Length(geometry)
返回曲面上的一个点 ST_PointOnSurface(geometry)
获取边界 ST_Boundary(geometry)
获取缓冲后的几何对象 ST_Buffer(geometry, double, [integer])
获取多几何对象的外接对象 ST_ConvexHull(geometry)
获取两个几何对象相交的部分 ST_Intersection(geometry, geometry)
将经度小于0的值加360使所有经度值在0-360间 ST_Shift_Longitude(geometry)
获取两个几何对象不相交的部分（A、B可互换） ST_SymDifference(geometry A, geometry B)
从A去除和B相交的部分后返回 ST_Difference(geometry A, geometry B)
返回两个几何对象的合并结果 ST_Union(geometry, geometry)
返回一系列几何对象的合并结果 ST_Union(geometry set)
用较少的内存和较长的时间完成合并操作，结果和ST_Union相同 ST_MemUnion(geometry set)
```

# 几何对象存取函数
```
获取几何对象的WKT描述 ST_AsText(geometry)
获取几何对象的WKB描述 ST_AsBinary(geometry)
获取几何对象的空间参考ID ST_SRID(geometry)
获取几何对象的维数 ST_Dimension(geometry)
获取几何对象的边界范围 ST_Envelope(geometry)
判断几何对象是否为空 ST_IsEmpty(geometry)
判断几何对象是否不包含特殊点（比如自相交） ST_IsSimple(geometry)
判断几何对象是否闭合 ST_IsClosed(geometry)
判断曲线是否闭合并且不包含特殊点 ST_IsRing(geometry)
获取多几何对象中的对象个数 ST_NumGeometries(geometry)
获取多几何对象中第N个对象 ST_GeometryN(geometry,int)
获取几何对象中的点个数 ST_NumPoints(geometry)
获取几何对象的第N个点 ST_PointN(geometry,integer)
获取多边形的外边缘 ST_ExteriorRing(geometry)
获取多边形内边界个数 ST_NumInteriorRings(geometry)
同上 ST_NumInteriorRing(geometry)
获取多边形的第N个内边界 ST_InteriorRingN(geometry,integer)
获取线的终点 ST_EndPoint(geometry)
获取线的起始点 ST_StartPoint(geometry)
获取几何对象的类型 GeometryType(geometry)
类似上，但是不检查M值，即POINTM对象会被判断为point ST_GeometryType(geometry)
获取点的X坐标 ST_X(geometry)
获取点的Y坐标 ST_Y(geometry)
获取点的Z坐标 ST_Z(geometry)
获取点的M值 ST_M(geometry)
```

# 几何对象构造函数
```
参考语义
Text：WKT
WKB：WKB
Geom:Geometry
M:Multi
Bd:BuildArea
Coll:Collection ST_GeomFromText(text,[])

ST_PointFromText(text,[])
ST_LineFromText(text,[])
ST_LinestringFromText(text,[])
ST_PolyFromText(text,[])
ST_PolygonFromText(text,[])
ST_MPointFromText(text,[])
ST_MLineFromText(text,[])
ST_MPolyFromText(text,[])
ST_GeomCollFromText(text,[])
ST_GeomFromWKB(bytea,[])
ST_GeometryFromWKB(bytea,[])
ST_PointFromWKB(bytea,[])
ST_LineFromWKB(bytea,[])
ST_LinestringFromWKB(bytea,[])
ST_PolyFromWKB(bytea,[])
ST_PolygonFromWKB(bytea,[])
ST_MPointFromWKB(bytea,[])
ST_MLineFromWKB(bytea,[])
ST_MPolyFromWKB(bytea,[])
ST_GeomCollFromWKB(bytea,[])
ST_BdPolyFromText(text WKT, integer SRID)
ST_BdMPolyFromText(text WKT, integer SRID)
```
