---
title: PostgreSQL 生成列 
date: 2021-03-15 15:35:49
tags: 
- PostgreSQL
- Generated column
categories: 
- PostgreSQL
- Generated column
top: 39 
description: 
password: 

---

PostgreSQL 生成列的三种使用方式

<!--more-->

# 自定义函数
```
CREATE OR REPLACE FUNCTION public.f2 (name text)
    RETURNS text
    LANGUAGE plpgsql
    IMMUTABLE
    AS $function$
BEGIN
    RETURN substring($1, 1, 10);
END;
$function$ 

CREATE TABLE people_gc_stored (
    name text,
    small_name text GENERATED ALWAYS AS (f2 (name)) STORED
);

```

# GENERATED ALWAYS
```
CREATE TABLE people_gc_stored1 (
    name text,
    small_name text GENERATED ALWAYS AS IDENTITY
);
```

# GENERATED DEFAULT
```
CREATE TABLE people_gc_stored2 (
    name text,
    small_name text GENERATED DEFAULT AS IDENTITY
);

```
