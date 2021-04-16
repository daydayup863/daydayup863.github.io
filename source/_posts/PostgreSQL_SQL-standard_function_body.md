---
title: SQL-standard function body 
date: 2021-04-16 15:21:25
tags: 
- PostgreSQL
- function
- SQL-standard
categories: 
- PostgreSQL
- function
- SQL-standard
top: 40
description: 
password: 

---

PostgreSQL 函数和储存过程支持SQL-standard function body

<!--more-->


# 特点

SQL-standard function body　在定义的时parse,并已expression nodes形式储存在pg_proc.prosqlbody字段, 因此在执行的时候不需要再次被parse.
由于在执行时不parse, 因此不支持多态的参数

```
    However, this form does not support polymorphic arguments, because
    there is no more parse analysis done at call time
    The function body is parsed at function definition time and stored as
    expression nodes in a new pg_proc column prosqlbody.  So at run time,
    no further parsing is required.

```

# 例子1
```
CREATE or replace  FUNCTION add(a integer, b integer) RETURNS integer as
$$
    select a + b;
$$LANGUAGE SQL;
```

# 例子2

```
CREATE FUNCTION add(a integer, b integer) RETURNS integer
LANGUAGE SQL
RETURN a + b;
```

# 例子3
```
CREATE or replace  PROCEDURE insert_data(a integer, b integer)
LANGUAGE SQL
as $$
INSERT INTO tbl VALUES (a);
INSERT INTO tbl VALUES (b);
$$;
```

# 例子4
```
CREATE PROCEDURE insert_data(a integer, b integer)
LANGUAGE SQL
BEGIN ATOMIC
  INSERT INTO tbl VALUES (a);
  INSERT INTO tbl VALUES (b);
END;
```

# duplicate function body
```
CREATE FUNCTION functest_S_xxx(x int) RETURNS int
    LANGUAGE SQL
    AS $$ SELECT x * 2 $$
    RETURN x * 3;
```


# polymorphic arguments not allowed in this form
```
CREATE FUNCTION functest_S_xx(x anyarray) RETURNS anyelement
    LANGUAGE SQL
    RETURN x[1];
```
# check reporting of parse-analysis errors
```
CREATE FUNCTION functest_S_xx(x date) RETURNS boolean
    LANGUAGE SQL
    RETURN x > 1;
```

# tricky parsing
```
CREATE FUNCTION functest_S_15(x int) RETURNS boolean
LANGUAGE SQL
BEGIN ATOMIC
    select case when x % 2 = 0 then true else false end;
END;

```

# 其它
```
CREATE FUNCTION functest_S_1(a text, b date) RETURNS boolean
    LANGUAGE SQL
    RETURN a = 'abcd' AND b > '2001-01-01';
CREATE FUNCTION functest_S_2(a text[]) RETURNS int
    RETURN a[1]::int;
CREATE FUNCTION functest_S_3() RETURNS boolean
    RETURN false;
CREATE FUNCTION functest_S_3a() RETURNS boolean
    BEGIN ATOMIC
        RETURN false;
    END;

CREATE FUNCTION functest_S_10(a text, b date) RETURNS boolean
    LANGUAGE SQL
    BEGIN ATOMIC
        SELECT a = 'abcd' AND b > '2001-01-01';
    END;

CREATE FUNCTION functest_S_13() RETURNS boolean
    BEGIN ATOMIC
        SELECT 1;
        SELECT false;
    END;
```
