---
title: PostgreSQL格式化SQL 
date: 2021-03-15 14:15:57
tags: 
- PostgreSQL
- sql
categories: 
- PostgreSQL
- sql
top: 36 
description: 
password: 

---

利用pg_get_viewdef完成SQL美化。

<!--more-->

```
mydb=# CREATE OR REPLACE FUNCTION format_sql(text)
RETURNS text AS
$$
   DECLARE
      v_ugly_string       ALIAS FOR $1;
      v_beauty            text;
      v_tmp_name          text;
   BEGIN
      -- let us create a unique view name
      v_tmp_name := 'temp_' || md5(v_ugly_string);
      EXECUTE 'CREATE TEMPORARY VIEW ' ||
      v_tmp_name || ' AS ' || v_ugly_string;

      -- the magic happens here
      SELECT pg_get_viewdef(v_tmp_name) INTO v_beauty;

      -- cleanup the temporary object
      EXECUTE 'DROP VIEW ' || v_tmp_name;
      RETURN v_beauty;
   EXCEPTION WHEN OTHERS THEN
      RAISE EXCEPTION 'you have provided an invalid string: % / %',
            sqlstate, sqlerrm;
   END;
$$ LANGUAGE 'plpgsql';
CREATE FUNCTION
mydb=# SELECT format_sql('SELECT * FROM
                  pg_tables UNION
                    ALL SELECT * FROM
          pg_tables');
-[ RECORD 1 ]-----------------------------
format_sql |  SELECT pg_tables.schemaname,+
           |     pg_tables.tablename,     +
           |     pg_tables.tableowner,    +
           |     pg_tables.tablespace,    +
           |     pg_tables.hasindexes,    +
           |     pg_tables.hasrules,      +
           |     pg_tables.hastriggers,   +
           |     pg_tables.rowsecurity    +
           |    FROM pg_tables            +
           | UNION ALL                    +
           |  SELECT pg_tables.schemaname,+
           |     pg_tables.tablename,     +
           |     pg_tables.tableowner,    +
           |     pg_tables.tablespace,    +
           |     pg_tables.hasindexes,    +
           |     pg_tables.hasrules,      +
           |     pg_tables.hastriggers,   +
           |     pg_tables.rowsecurity    +
           |    FROM pg_tables;

mydb=#


```

