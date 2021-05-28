---
title: PostgreSQL函数/存储过程返回多个游标
date: 2021-03-12 14:15:38
tags: 
- PostgreSQL
- cursor
- refcursor
categories:
- PostgreSQL
- cursor
- refcursor
top: 35 
description: 
password: 

---

# function 返回多个游标
```
CREATE OR REPLACE FUNCTION show_cities_multiple() RETURNS SETOF refcursor AS $$
 DECLARE
   ref1 refcursor;           -- Declare cursor variables
   ref2 refcursor;
 BEGIN
   OPEN ref1 FOR SELECT 1;   -- Open the first cursor
   RETURN NEXT ref1;                                                                              -- Return the cursor to the caller

   OPEN ref2 FOR SELECT 2;   -- Open the second cursor
   RETURN NEXT ref2;                                                                              -- Return the cursor to the caller
 END;
 $$ LANGUAGE plpgsql;
```

<!--more-->

# procedure 返回多个游标
```
CREATE OR REPLACE PROCEDURE public.test_p1(INOUT ref refcursor[])
 LANGUAGE plpgsql
AS $procedure$
declare ref0 refcursor; ref1 refcursor;
begin
open ref0 for select 1; ref[0] = ref0;
open ref1 for select 2; ref[1] = ref1;
end;
$procedure$
```
