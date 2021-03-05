---
title: PostgreSQL有用的SQL
date: 2020-07-31 18:19:15
tags: [PostgreSQL, DBA, SQL]
categories: 数据库
top: 10

---

记录工作中常用到的PostgreSQL常用SQL， tps, qps 等等.

<!-- more -->

# 主库查询主从延迟
```
 SELECT pg_stat_replication.application_name,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), pg_stat_replication.replay_lsn)) AS diff
   FROM pg_stat_replication;
```

# 从库查询主从延迟
```
SELECT
    CASE
        WHEN pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn() THEN 0::double precision
        ELSE date_part('epoch'::text, now() - pg_last_xact_replay_timestamp())
    END AS log_delay;
```

# 按query分组查询活跃SQL数量
```
 SELECT pg_stat_activity.query,
    count(1) AS count
   FROM pg_stat_activity
  WHERE pg_stat_activity.state <> 'idle'::text AND pg_stat_activity.pid <> pg_backend_pid() AND pg_stat_activity.query <> ''::text AND pg_stat_activity.query !~ '^DISCARD|^SET extra_float_digits = 3'::text
  GROUP BY pg_stat_activity.query
 HAVING count(1) > 1
  ORDER BY (count(1)) DESC
 LIMIT 10;
```

# 查询锁
```
select
      wait.pid,
      wait.state,
      wait.datname,
      (select string_agg(distinct transactionid::text, ',') from pg_locks where pid = wait.pid and locktype = 'transactionid' and transactionid::text <> wait.transactionid::text),
      wait.virtualxid,
      wait.locktype,
      wait.usename,
      wait.application_name,
      wait.client_addr,
      wait.wait_event_type,
      wait.wait_event,
      wait.query_start,
      wait.query,

      wait.relation,
      wait.datname || '.' || d.nspname || '.' || c.relname as relname,
      granted.relation,

      granted.pid              as waitfor_pid,
      granted.state            as waitfor_state,
      granted.transactionid    as waitfor_transactionid,
      granted.virtualxid       as waitfor_virtualxid,
      granted.locktype         as waitfor_locktype,
      granted.usename          as waitfor_usename,
      granted.client_addr      as waitfor_client_addr,
      granted.application_name as waitfor_application_name,
      granted.wait_event_type  as waitfor_wait_event_type,
      granted.wait_event       as waitfor_wait_event,
      granted.query_start      as waitfor_query_start,
      granted.query            as waitfor_query,
      now() - granted.query_start  as waitfor_time

from
    (select
          a.pid,
          a.state,
          b.transactionid,
          b.virtualxid,
          b.locktype,
          b.relation,
          b.page,
          b.tuple,
          a.usename,
          a.datname,
          a.application_name,
          a.client_addr,
          a.wait_event_type,
          a.wait_event,
          a.query_start,
          a.query
     from
          pg_stat_activity a,
          pg_locks b
     where
          a.wait_event_type is not null
          and a.pid = b.pid
          and granted = 'f'
    ) wait
join
    (select
          b.pid,
          b.state,
          a.transactionid,
          a.virtualxid,
          a.locktype,
          a.relation,
          a.page,
          a.tuple,
          b.usename,
          b.datname,
          b.application_name,
          b.client_addr,
          b.wait_event_type,
          b.wait_event,
          b.query_start,
          b.query
    from
        pg_locks a,
        pg_stat_activity b
    where
        a.pid = b.pid
        and a.granted = 't'
    ) granted
on (
    ( wait.locktype = 'transactionid'
    and granted.locktype = 'transactionid'
    and wait.transactionid = granted.transactionid )
    or
    ( wait.locktype = 'relation'
    and granted.locktype = 'relation'
    and wait.relation = granted.relation
    )
    or
    ( wait.locktype = 'virtualxid'
    and granted.locktype = 'virtualxid'
    and wait.virtualxid = granted.virtualxid )
    or
    ( wait.locktype = 'tuple'
    and granted.locktype = 'tuple'
    and wait.relation = granted.relation
    and wait.page = granted.page
    and wait.tuple = granted.tuple )
)
left join
    pg_class c
on ( c.relfilenode = wait.relation )
left join
    pg_namespace d
on ( c.relnamespace = d.oid )
order by
    granted.query_start
;
```

# 查询表大小, 按占用大小排序
```
SELECT
    t.spcname,
    (n.nspname::text || '.'::text) || c.relname::text AS relation,
    pg_size_pretty(pg_total_relation_size(c.oid::regclass)) AS total_table_size,
    pg_size_pretty(pg_relation_size(c.oid::regclass)) AS table_only_size,
    pg_size_pretty(pg_indexes_size(c.oid::regclass)) AS total_index_size
FROM
    pg_class c
    LEFT JOIN pg_tablespace t ON c.reltablespace = t.oid
    LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE (n.nspname <> ALL (ARRAY['pg_catalog'::name, 'information_schema'::name, 'pg_toast'::name]))
    AND c.relkind = 'r'::"char"
ORDER BY
    (pg_total_relation_size(c.oid::regclass)) DESC;
```



# 找出重复的index
```
SELECT
    relname,
    (array_agg(idx))[1] idx1,
    pg_get_indexdef((array_agg(idx))[1]) idx1_def,
    (array_agg(idx))[2] idx2,
    pg_get_indexdef((array_agg(idx))[2]) idx2_def,
    (array_agg(idx))[3] idx3,
    pg_get_indexdef((array_agg(idx))[3]) idx3_def
FROM (
    SELECT
        indrelid::regclass AS relname,
        indexrelid::regclass AS idx,
        (indrelid::text || indclass::text || indkey::text || COALESCE(indexprs::text, '') || COALESCE(indpred::text, '')) AS KEY
    FROM
        pg_index) sub
GROUP BY
    relname,
    KEY
HAVING
    count(*) > 1;
```

# 自定义操作符兼容Oracle varchar = int
```
mydb=# create table test(info varchar);
CREATE TABLE
mydb=# insert into test select '1';
INSERT 0 1
mydb=# insert into test select '2';
INSERT 0 1
mydb=# insert into test select '3';
INSERT 0 1
mydb=# select * from test;
 info
------
 1
 2
 3
(3 rows)

mydb=# select * from test where info = 1;
ERROR:  operator does not exist: character varying = integer
LINE 1: select * from test where info = 1;
                                      ^
HINT:  No operator matches the given name and argument types. You might need to add explicit type casts.
mydb=# CREATE OR REPLACE FUNCTION equal(character varying, integer) RETURNS boolean AS $function$ select cast($1 as text) = cast($2 as text) $function$ LANGUAGE sql;
CREATE FUNCTION
mydb=# create operator = (leftarg = varchar, rightarg = int, procedure = equal, commutator = =);
CREATE OPERATOR
mydb=# select * from test where info = 1;
 info
------
 1
(1 row)

mydb=#

```

# 兼容instr 
```
mydb=# select instr('helloworld', 'l', 2, 2);
 instr
-------
     4
(1 row)

mydb=# select instr('helloworld', 'l', 3, 2);
 instr
-------
     4
(1 row)

mydb=# select instr('helloworld', 'l', -1, 1);
 instr
-------
     9
(1 row)

mydb=# select instr('helloworld', 'l', -2, 1);
 instr
-------
     9
(1 row)

mydb=# \sf instr
CREATE OR REPLACE FUNCTION public.instr(str text, sub text, startpos integer DEFAULT 1, occurrence integer DEFAULT 1)
 RETURNS integer
 LANGUAGE plpgsql
AS $function$
declare
    tail text;
    shift int;
    pos int;
    i int;
begin
    shift:= 0;
    if startpos = 0 or occurrence <= 0 then
        return 0;
    end if;
    if startpos < 0 then
        str:= reverse(str);
        sub:= reverse(sub);
        pos:= -startpos;
    else
        pos:= startpos;
    end if;
    for i in 1..occurrence loop
        shift:= shift+ pos;
        tail:= substr(str, shift);
        pos:= strpos(tail, sub);
        if pos = 0 then
            return 0;
        end if;
    end loop;
    if startpos > 0 then
        return pos+ shift- 1;
    else
        return length(str)- length(sub)- pos- shift+ 3;
    end if;
end $function$
mydb=#

```

# procedure 异常处理
```
mydb=# CREATE OR REPLACE PROCEDURE f1()
 LANGUAGE plpgsql
AS $procedure$
declare
  my_ex_state text;
  my_ex_message text;
  my_ex_detail text;
  my_ex_hint text;
  my_ex_ctx text;
begin
  begin
    raise notice 'A';
  exception when others then
    raise notice 'C';
    GET STACKED DIAGNOSTICS
      my_ex_state   = RETURNED_SQLSTATE,
      my_ex_message = MESSAGE_TEXT,
      my_ex_detail  = PG_EXCEPTION_DETAIL,
      my_ex_hint    = PG_EXCEPTION_HINT,
      my_ex_ctx     = PG_EXCEPTION_CONTEXT
    ;
    raise notice '% % % % %', my_ex_state, my_ex_message, my_ex_detail, my_ex_hint, my_ex_ctx;
  end;

  commit;

  begin
    raise notice 'B';
  exception when others then
    raise notice 'C';
    GET STACKED DIAGNOSTICS
      my_ex_state   = RETURNED_SQLSTATE,
      my_ex_message = MESSAGE_TEXT,
      my_ex_detail  = PG_EXCEPTION_DETAIL,
      my_ex_hint    = PG_EXCEPTION_HINT,
      my_ex_ctx     = PG_EXCEPTION_CONTEXT
    ;
    raise notice '% % % % %', my_ex_state, my_ex_message, my_ex_detail, my_ex_hint, my_ex_ctx;
  end;
end;
$procedure$
;

```

# 找下一个周末
```
postgres=# select next_day('2020-12-22'::date, 7);
  next_day
------------
 2020-12-26
(1 row)

postgres=# select next_day('2020-12-21'::date, 7);
  next_day
------------
 2020-12-26
(1 row)

postgres=# select next_day('2020-12-20'::date, 7);
  next_day
------------
 2020-12-26
(1 row)

```

# PostgreSQL只迁移FUNCTION
```
pg_dump -h localhost -U jintao -Fc -s -f db_dump mydb
more db_dump
pg_restore -l db_dump | grep FUNCTION > function_list
cat function_list
pg_restore -h localhost -U postgres -d postgres -L function_list db_dump
```

# pageinspect 查看xmax的状态

```
SELECT lp,
       t_ctid AS ctid,
       t_xmin AS xmin,
       t_xmax AS xmax,
       (t_infomask & 128)::boolean AS xmax_is_lock,
       (t_infomask & 1024)::boolean AS xmax_committed,
       (t_infomask & 2048)::boolean AS xmax_rolled_back,
       (t_infomask & 4096)::boolean AS xmax_multixact,
       t_attrs[1] AS p_id,
       t_attrs[2] AS p_val
FROM heap_page_item_attrs(
        get_raw_page('parent', 0),
        'parent'
     );
```

# PostgreSQL9.1主从延迟查询
```
postgres=# \sf pg_xlog_location_diff
CREATE OR REPLACE FUNCTION pg_catalog.pg_xlog_location_diff(text, text)
 RETURNS numeric
 LANGUAGE plpgsql
AS $function$
    DECLARE
       offset1 text;
       offset2 text;
       xlog1 text;
       xlog2 text;
       SQL text;
       diff text;
    BEGIN
       /* Extract the Offset and xlog from input in
          offset and xlog variables */

       offset1=split_part($1,'/',2);
         xlog1=split_part($1,'/',1);
       offset2=split_part($2,'/',2);
         xlog2=split_part($2,'/',1);

       /* Prepare SQL query for calculation based on following formula
         (FF000000 * xlog + offset) - (FF000000 * xlog + offset)
         which gives value in hexadecimal. Since, hexadecimal calculation is cumbersome
         so convert into decimal and then calculate the difference */

       SQL='SELECT (x'''||'FF000000'||'''::bigint * x'''||xlog1||'''::bigint
                                +  x'''||offset1||'''::bigint)'||'
                -
                   (x'''||'FF000000'||'''::bigint * x'''||xlog2||'''::bigint
                                +  x'''||offset2||'''::bigint)';
       EXECUTE SQL into diff;

       /* Return the value in numeric by explicit casting  */

       RETURN diff::numeric;
    END;
 $function$
postgres=# SELECT
    application_name,
    pg_xlog_location_diff(pg_current_xlog_location(), replay_location) as diff
FROM
    pg_stat_replication
;
 application_name | diff
------------------+-------
 walreceiver      | 18272
(1 row)

```

# 获取cluster初始化的时间
```
$ pg_controldata |grep -i system
Database system identifier:           6888133981158752630
$ psql
psql (14devel)
Type "help" for help.

mydb=# SELECT to_timestamp(((6888133981158752630>>32) & (2^32 -1)::bigint));
      to_timestamp
------------------------
 2020-10-27 11:17:48+08
(1 row)

mydb=#

```

# tuple-internal详解
```
https://pgconf.ru/media/2016/05/13/tuple-internals.pdf
```

# PostgreSQL clog最大大小
```
#define CLOG_BITS_PER_XACT      2   //2bit一个事物
#define CLOG_XACTS_PER_BYTE 4       //1Byte4个事物
#define CLOG_XACTS_PER_PAGE (BLCKSZ * CLOG_XACTS_PER_BYTE)

500M，内存中处理完全够效率, 想想要是64位xid, 光clog大小就不一定多大了.

clog总大小约500MB, 每个clog文件256kb, 最多2097152个clog文件.

mydb=# select 2^31/4/256;
 ?column?
----------
  2097152
(1 row)

mydb=# select pg_size_pretty((2^31/4)::bigint);
 pg_size_pretty
----------------
 512 MB
(1 row)

mydb=#

```

# bytea转为原始字符
```
postgres=# create table t_bytea(info bytea);
CREATE TABLE
postgres=# insert into t_bytea select 'hello';
INSERT 0 1
postgres=# insert into t_bytea select '我的家乡';
INSERT 0 1
postgres=# select * from t_bytea ;
            info
----------------------------
 \x68656c6c6f
 \xe68891e79a84e5aeb6e4b9a1
(2 rows)

postgres=# select encode(info, 'escape') from t_bytea ;
                      encode
--------------------------------------------------
 hello
 \346\210\221\347\232\204\345\256\266\344\271\241
(2 rows)

postgres=# select convert_from(info, 'UTF8') from t_bytea ;
 convert_from
--------------
 hello
 我的家乡
(2 rows)

```

```
create schema dba;
```
#  tps
```
create or replace procedure dba.tps() as $$
declare
  v1 int8;
  v2 int8;
begin
  select txid_snapshot_xmax(txid_current_snapshot()) into v1;
  commit;
  perform pg_sleep(1);
  select txid_snapshot_xmax(txid_current_snapshot()) into v2;
  commit;
  raise notice 'tps: %', v2-v1;
end;
$$ language plpgsql ;
```
# qps

用select sum(calls) s from pg_stat_statements(false) 而不用select sum(calls) s from pg_stat_statements
不需要具体的query, 查询效率好很多.
```
with
a as (select sum(calls) s from pg_stat_statements(false)),   
b as (select sum(calls) s from pg_stat_statements(false) , pg_sleep(1))   
select   
b.s-a.s          -- QPS  
from a,b;
```

#  查询没有使用过的大于1MB的索引 top 10
```
CREATE VIEW dba.top10notusedidx AS
SELECT
    pg_size_pretty(pg_relation_size(indexrelid)),
    *
FROM
    pg_stat_all_indexes
WHERE
    pg_relation_size(indexrelid) >= 1024000
    AND (idx_scan = 0
        OR idx_tup_read = 0
        OR idx_tup_fetch = 0)
    AND schemaname NOT IN ('pg_toast', 'pg_catalog')
ORDER BY
    pg_relation_size(indexrelid) DESC
LIMIT 10;
```
注意, PK、UK如果只是用于约束, 可能不会被统计计数,但是不能删掉) 

# 查询没有使用过的大于1MB的表 top 10
```
CREATE VIEW dba.top10notusedtab AS
SELECT
    pg_size_pretty(pg_relation_size(relid)),
    *
FROM
    pg_stat_all_tables
WHERE
    pg_relation_size(relid) >= 1024000
    AND seq_scan = 0
    AND idx_scan = 0
    AND schemaname NOT IN ('pg_toast', 'pg_catalog', 'information_schema')
ORDER BY
    pg_relation_size(relid) DESC
LIMIT 10;
```
# 查询热表top 10
```
CREATE VIEW dba.top10hottab AS
SELECT
    pg_size_pretty(pg_relation_size(relid)),
    *
FROM
    pg_stat_all_tables
WHERE
    schemaname NOT IN ('pg_toast', 'pg_catalog', 'information_schema')
ORDER BY
    seq_scan + idx_scan DESC,
    pg_relation_size(relid) DESC
LIMIT 10;
```
# 在standby节点执行， 接收wal的速度。
```
CREATE OR REPLACE PROCEDURE dba.wal_receive_bw()
 LANGUAGE plpgsql
AS $procedure$
declare
  v1 pg_lsn;
  v2 pg_lsn;
begin
  select pg_last_wal_receive_lsn() into v1;
  commit;
  perform pg_sleep(1);
  select pg_last_wal_receive_lsn() into v2;
  commit;
  raise notice 'wal receive bw: %/s', pg_size_pretty(pg_wal_lsn_diff(v2,v1));
end;
$procedure$;
```
# 在standby节点执行， replay wal的速度。
```
CREATE OR REPLACE PROCEDURE dba.wal_replay_bw()
 LANGUAGE plpgsql
AS $procedure$
declare
  v1 pg_lsn;
  v2 pg_lsn;
begin
  select pg_last_wal_replay_lsn() into v1;
  commit;
  perform pg_sleep(1);
  select pg_last_wal_replay_lsn() into v2;
  commit;
  raise notice 'wal replay bw: %/s', pg_size_pretty(pg_wal_lsn_diff(v2,v1));
end;
$procedure$; 
```

# 查询膨胀空间top 10的表
```
create view dba.top10bloatsizetable as  
SELECT  
  current_database() AS db, schemaname, tablename, reltuples::bigint AS tups, relpages::bigint AS pages, otta,  
  ROUND(CASE WHEN otta=0 OR sml.relpages=0 OR sml.relpages=otta THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat,  
  CASE WHEN relpages < otta THEN 0 ELSE relpages::bigint - otta END AS wastedpages,  
  CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,  
  CASE WHEN relpages < otta THEN '0 bytes'::text ELSE pg_size_pretty((bs*(relpages-otta))::bigint) END AS wastedsize,  
  iname, ituples::bigint AS itups, ipages::bigint AS ipages, iotta,  
  ROUND(CASE WHEN iotta=0 OR ipages=0 OR ipages=iotta THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat,  
  CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages,  
  CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes,  
  CASE WHEN ipages < iotta THEN '0 bytes' ELSE pg_size_pretty((bs*(ipages-iotta))::bigint) END AS wastedisize,  
  pg_size_pretty(CASE WHEN relpages < otta THEN  
    CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta::bigint) END  
    ELSE CASE WHEN ipages < iotta THEN bs*(relpages-otta::bigint)  
      ELSE bs*(relpages-otta::bigint + ipages-iotta::bigint) END  
  END) AS totalwastedbytes  
FROM (  
  SELECT  
    nn.nspname AS schemaname,  
    cc.relname AS tablename,  
    COALESCE(cc.reltuples,0) AS reltuples,  
    COALESCE(cc.relpages,0) AS relpages,  
    COALESCE(bs,0) AS bs,  
    COALESCE(CEIL((cc.reltuples*((datahdr+ma-  
      (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)),0) AS otta,  
    COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,  
    COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta -- very rough approximation, assumes all cols  
  FROM  
     pg_class cc  
  JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname <> 'information_schema'  
  LEFT JOIN  
  (  
    SELECT  
      ma,bs,foo.nspname,foo.relname,  
      (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,  
      (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2  
    FROM (  
      SELECT  
        ns.nspname, tbl.relname, hdr, ma, bs,  
        SUM((1-coalesce(null_frac,0))*coalesce(avg_width, 2048)) AS datawidth,  
        MAX(coalesce(null_frac,0)) AS maxfracsum,  
        hdr+(  
          SELECT 1+count(*)/8  
          FROM pg_stats s2  
          WHERE null_frac<>0 AND s2.schemaname = ns.nspname AND s2.tablename = tbl.relname  
        ) AS nullhdr  
      FROM pg_attribute att  
      JOIN pg_class tbl ON att.attrelid = tbl.oid  
      JOIN pg_namespace ns ON ns.oid = tbl.relnamespace  
      LEFT JOIN pg_stats s ON s.schemaname=ns.nspname  
      AND s.tablename = tbl.relname  
      AND s.inherited=false  
      AND s.attname=att.attname,  
      (  
        SELECT  
          (SELECT current_setting('block_size')::numeric) AS bs,  
            CASE WHEN SUBSTRING(SPLIT_PART(v, ' ', 2) FROM '#"[0-9]+.[0-9]+#"%' for '#')  
              IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,  
          CASE WHEN v ~ 'mingw32' OR v ~ '64-bit' THEN 8 ELSE 4 END AS ma  
        FROM (SELECT version() AS v) AS foo  
      ) AS constants  
      WHERE att.attnum > 0 AND tbl.relkind='r'  
      GROUP BY 1,2,3,4,5  
    ) AS foo  
  ) AS rs  
  ON cc.relname = rs.relname AND nn.nspname = rs.nspname  
  LEFT JOIN pg_index i ON indrelid = cc.oid  
  LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid  
) AS sml order by wastedbytes desc limit 5;  
```

# 查询膨胀空间top 10的索引
```
create view dba.top10bloatsizeindex as  
SELECT  
  current_database() AS db, schemaname, tablename, reltuples::bigint AS tups, relpages::bigint AS pages, otta,  
  ROUND(CASE WHEN otta=0 OR sml.relpages=0 OR sml.relpages=otta THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat,  
  CASE WHEN relpages < otta THEN 0 ELSE relpages::bigint - otta END AS wastedpages,  
  CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,  
  CASE WHEN relpages < otta THEN '0 bytes'::text ELSE pg_size_pretty((bs*(relpages-otta))::bigint) END AS wastedsize,  
  iname, ituples::bigint AS itups, ipages::bigint AS ipages, iotta,  
  ROUND(CASE WHEN iotta=0 OR ipages=0 OR ipages=iotta THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat,  
  CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages,  
  CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes,  
  CASE WHEN ipages < iotta THEN '0 bytes' ELSE pg_size_pretty((bs*(ipages-iotta))::bigint) END AS wastedisize,  
  pg_size_pretty(CASE WHEN relpages < otta THEN  
    CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta::bigint) END  
    ELSE CASE WHEN ipages < iotta THEN bs*(relpages-otta::bigint)  
      ELSE bs*(relpages-otta::bigint + ipages-iotta::bigint) END  
  END) AS totalwastedbytes  
FROM (  
  SELECT  
    nn.nspname AS schemaname,  
    cc.relname AS tablename,  
    COALESCE(cc.reltuples,0) AS reltuples,  
    COALESCE(cc.relpages,0) AS relpages,  
    COALESCE(bs,0) AS bs,  
    COALESCE(CEIL((cc.reltuples*((datahdr+ma-  
      (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)),0) AS otta,  
    COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,  
    COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta -- very rough approximation, assumes all cols  
  FROM  
     pg_class cc  
  JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname <> 'information_schema'  
  LEFT JOIN  
  (  
    SELECT  
      ma,bs,foo.nspname,foo.relname,  
      (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,  
      (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2  
    FROM (  
      SELECT  
        ns.nspname, tbl.relname, hdr, ma, bs,  
        SUM((1-coalesce(null_frac,0))*coalesce(avg_width, 2048)) AS datawidth,  
        MAX(coalesce(null_frac,0)) AS maxfracsum,  
        hdr+(  
          SELECT 1+count(*)/8  
          FROM pg_stats s2  
          WHERE null_frac<>0 AND s2.schemaname = ns.nspname AND s2.tablename = tbl.relname  
        ) AS nullhdr  
      FROM pg_attribute att  
      JOIN pg_class tbl ON att.attrelid = tbl.oid  
      JOIN pg_namespace ns ON ns.oid = tbl.relnamespace  
      LEFT JOIN pg_stats s ON s.schemaname=ns.nspname  
      AND s.tablename = tbl.relname  
      AND s.inherited=false  
      AND s.attname=att.attname,  
      (  
        SELECT  
          (SELECT current_setting('block_size')::numeric) AS bs,  
            CASE WHEN SUBSTRING(SPLIT_PART(v, ' ', 2) FROM '#"[0-9]+.[0-9]+#"%' for '#')  
              IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,  
          CASE WHEN v ~ 'mingw32' OR v ~ '64-bit' THEN 8 ELSE 4 END AS ma  
        FROM (SELECT version() AS v) AS foo  
      ) AS constants  
      WHERE att.attnum > 0 AND tbl.relkind='r'  
      GROUP BY 1,2,3,4,5  
    ) AS foo  
  ) AS rs  
  ON cc.relname = rs.relname AND nn.nspname = rs.nspname  
  LEFT JOIN pg_index i ON indrelid = cc.oid  
  LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid  
) AS sml order by wastedibytes desc limit 5;  
```
# 查询膨胀比例top 10的表(浪费空间大于10MB的表)
```
create view dba.top10bloatratiotable as  
SELECT  
  current_database() AS db, schemaname, tablename, reltuples::bigint AS tups, relpages::bigint AS pages, otta,  
  ROUND(CASE WHEN otta=0 OR sml.relpages=0 OR sml.relpages=otta THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat,  
  CASE WHEN relpages < otta THEN 0 ELSE relpages::bigint - otta END AS wastedpages,  
  CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,  
  CASE WHEN relpages < otta THEN '0 bytes'::text ELSE pg_size_pretty((bs*(relpages-otta))::bigint) END AS wastedsize,  
  iname, ituples::bigint AS itups, ipages::bigint AS ipages, iotta,  
  ROUND(CASE WHEN iotta=0 OR ipages=0 OR ipages=iotta THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat,  
  CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages,  
  CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes,  
  CASE WHEN ipages < iotta THEN '0 bytes' ELSE pg_size_pretty((bs*(ipages-iotta))::bigint) END AS wastedisize,  
  pg_size_pretty(CASE WHEN relpages < otta THEN  
    CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta::bigint) END  
    ELSE CASE WHEN ipages < iotta THEN bs*(relpages-otta::bigint)  
      ELSE bs*(relpages-otta::bigint + ipages-iotta::bigint) END  
  END) AS totalwastedbytes  
FROM (  
  SELECT  
    nn.nspname AS schemaname,  
    cc.relname AS tablename,  
    COALESCE(cc.reltuples,0) AS reltuples,  
    COALESCE(cc.relpages,0) AS relpages,  
    COALESCE(bs,0) AS bs,  
    COALESCE(CEIL((cc.reltuples*((datahdr+ma-  
      (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)),0) AS otta,  
    COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,  
    COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta -- very rough approximation, assumes all cols  
  FROM  
     pg_class cc  
  JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname <> 'information_schema'  
  LEFT JOIN  
  (  
    SELECT  
      ma,bs,foo.nspname,foo.relname,  
      (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,  
      (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2  
    FROM (  
      SELECT  
        ns.nspname, tbl.relname, hdr, ma, bs,  
        SUM((1-coalesce(null_frac,0))*coalesce(avg_width, 2048)) AS datawidth,  
        MAX(coalesce(null_frac,0)) AS maxfracsum,  
        hdr+(  
          SELECT 1+count(*)/8  
          FROM pg_stats s2  
          WHERE null_frac<>0 AND s2.schemaname = ns.nspname AND s2.tablename = tbl.relname  
        ) AS nullhdr  
      FROM pg_attribute att  
      JOIN pg_class tbl ON att.attrelid = tbl.oid  
      JOIN pg_namespace ns ON ns.oid = tbl.relnamespace  
      LEFT JOIN pg_stats s ON s.schemaname=ns.nspname  
      AND s.tablename = tbl.relname  
      AND s.inherited=false  
      AND s.attname=att.attname,  
      (  
        SELECT  
          (SELECT current_setting('block_size')::numeric) AS bs,  
            CASE WHEN SUBSTRING(SPLIT_PART(v, ' ', 2) FROM '#"[0-9]+.[0-9]+#"%' for '#')  
              IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,  
          CASE WHEN v ~ 'mingw32' OR v ~ '64-bit' THEN 8 ELSE 4 END AS ma  
        FROM (SELECT version() AS v) AS foo  
      ) AS constants  
      WHERE att.attnum > 0 AND tbl.relkind='r'  
      GROUP BY 1,2,3,4,5  
    ) AS foo  
  ) AS rs  
  ON cc.relname = rs.relname AND nn.nspname = rs.nspname  
  LEFT JOIN pg_index i ON indrelid = cc.oid  
  LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid  
) AS sml   
where (CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END) >= 10240000  
order by tbloat desc,wastedbytes desc limit 5;  
```
# 查询膨胀比例top 10的索引(浪费空间大于10MB的索引)
```
create view dba.top10bloatratioindex as  
SELECT  
  current_database() AS db, schemaname, tablename, reltuples::bigint AS tups, relpages::bigint AS pages, otta,  
  ROUND(CASE WHEN otta=0 OR sml.relpages=0 OR sml.relpages=otta THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat,  
  CASE WHEN relpages < otta THEN 0 ELSE relpages::bigint - otta END AS wastedpages,  
  CASE WHEN relpages < otta THEN 0 ELSE bs*(sml.relpages-otta)::bigint END AS wastedbytes,  
  CASE WHEN relpages < otta THEN '0 bytes'::text ELSE pg_size_pretty((bs*(relpages-otta))::bigint) END AS wastedsize,  
  iname, ituples::bigint AS itups, ipages::bigint AS ipages, iotta,  
  ROUND(CASE WHEN iotta=0 OR ipages=0 OR ipages=iotta THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat,  
  CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages,  
  CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes,  
  CASE WHEN ipages < iotta THEN '0 bytes' ELSE pg_size_pretty((bs*(ipages-iotta))::bigint) END AS wastedisize,  
  pg_size_pretty(CASE WHEN relpages < otta THEN  
    CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta::bigint) END  
    ELSE CASE WHEN ipages < iotta THEN bs*(relpages-otta::bigint)  
      ELSE bs*(relpages-otta::bigint + ipages-iotta::bigint) END  
  END) AS totalwastedbytes  
FROM (  
  SELECT  
    nn.nspname AS schemaname,  
    cc.relname AS tablename,  
    COALESCE(cc.reltuples,0) AS reltuples,  
    COALESCE(cc.relpages,0) AS relpages,  
    COALESCE(bs,0) AS bs,  
    COALESCE(CEIL((cc.reltuples*((datahdr+ma-  
      (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)),0) AS otta,  
    COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages,  
    COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta -- very rough approximation, assumes all cols  
  FROM  
     pg_class cc  
  JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname <> 'information_schema'  
  LEFT JOIN  
  (  
    SELECT  
      ma,bs,foo.nspname,foo.relname,  
      (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,  
      (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2  
    FROM (  
      SELECT  
        ns.nspname, tbl.relname, hdr, ma, bs,  
        SUM((1-coalesce(null_frac,0))*coalesce(avg_width, 2048)) AS datawidth,  
        MAX(coalesce(null_frac,0)) AS maxfracsum,  
        hdr+(  
          SELECT 1+count(*)/8  
          FROM pg_stats s2  
          WHERE null_frac<>0 AND s2.schemaname = ns.nspname AND s2.tablename = tbl.relname  
        ) AS nullhdr  
      FROM pg_attribute att  
      JOIN pg_class tbl ON att.attrelid = tbl.oid  
      JOIN pg_namespace ns ON ns.oid = tbl.relnamespace  
      LEFT JOIN pg_stats s ON s.schemaname=ns.nspname  
      AND s.tablename = tbl.relname  
      AND s.inherited=false  
      AND s.attname=att.attname,  
      (  
        SELECT  
          (SELECT current_setting('block_size')::numeric) AS bs,  
            CASE WHEN SUBSTRING(SPLIT_PART(v, ' ', 2) FROM '#"[0-9]+.[0-9]+#"%' for '#')  
              IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr,  
          CASE WHEN v ~ 'mingw32' OR v ~ '64-bit' THEN 8 ELSE 4 END AS ma  
        FROM (SELECT version() AS v) AS foo  
      ) AS constants  
      WHERE att.attnum > 0 AND tbl.relkind='r'  
      GROUP BY 1,2,3,4,5  
    ) AS foo  
  ) AS rs  
  ON cc.relname = rs.relname AND nn.nspname = rs.nspname  
  LEFT JOIN pg_index i ON indrelid = cc.oid  
  LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid  
) AS sml   
where (CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END) >= 10240000  
order by ibloat desc,wastedibytes desc limit 5; 
```
# 查询序列距离最大值的范围
```
create view dba.seqs as select max_value-last_value,* from pg_sequences order by max_value-last_value ;
```
# freeze风暴预测相关的3个视图
```
create view dba.v_freeze as    
select     
  e.*,     
  a.*     
from    
(select     
  current_setting('autovacuum_freeze_max_age')::int as v1,            -- 如果表的事务ID年龄大于该值, 即使未开启autovacuum也会强制触发FREEZE, 并告警Preventing Transaction ID Wraparound Failures    
  current_setting('autovacuum_multixact_freeze_max_age')::int as v2,  -- 如果表的并行事务ID年龄大于该值, 即使未开启autovacuum也会强制触发FREEZE, 并告警Preventing Transaction ID Wraparound Failures    
  current_setting('vacuum_freeze_min_age')::int as v3,                -- 手动或自动垃圾回收时, 如果记录的事务ID年龄大于该值, 将被FREEZE    
  current_setting('vacuum_multixact_freeze_min_age')::int as v4,      -- 手动或自动垃圾回收时, 如果记录的并行事务ID年龄大于该值, 将被FREEZE    
  current_setting('vacuum_freeze_table_age')::int as v5,              -- 手动垃圾回收时, 如果表的事务ID年龄大于该值, 将触发FREEZE. 该参数的上限值为 %95 autovacuum_freeze_max_age    
  current_setting('vacuum_multixact_freeze_table_age')::int as v6,    -- 手动垃圾回收时, 如果表的并行事务ID年龄大于该值, 将触发FREEZE. 该参数的上限值为 %95 autovacuum_multixact_freeze_max_age    
  current_setting('autovacuum_vacuum_cost_delay') as v7,              -- 自动垃圾回收时, 每轮回收周期后的一个休息时间, 主要防止垃圾回收太耗资源. -1 表示沿用vacuum_cost_delay的设置    
  current_setting('autovacuum_vacuum_cost_limit') as v8,              -- 自动垃圾回收时, 每轮回收周期设多大限制, 限制由vacuum_cost_page_hit,vacuum_cost_page_missvacuum_cost_page_dirty参数以及周期内的操作决定. -1 表示沿用vacuum_cost_limit的设置    
  current_setting('vacuum_cost_delay') as v9,                         -- 手动垃圾回收时, 每轮回收周期后的一个休息时间, 主要防止垃圾回收太耗资源.    
  current_setting('vacuum_cost_limit') as v10,                        -- 手动垃圾回收时, 每轮回收周期设多大限制, 限制由vacuum_cost_page_hit,vacuum_cost_page_missvacuum_cost_page_dirty参数以及周期内的操作决定.    
  current_setting('autovacuum') as autovacuum                         -- 是否开启自动垃圾回收    
) a,     
LATERAL (   -- LATERAL 允许你在这个SUBQUERY中直接引用前面的table, subquery中的column     
select     
pg_size_pretty(pg_total_relation_size(oid)) sz,   -- 表的大小(含TOAST, 索引)    
oid::regclass as reloid,    -- 表名(物化视图)    
relkind,                    -- r=表, m=物化视图    
coalesce(    
  least(    
    substring(reloptions::text, 'autovacuum_freeze_max_age=(\d+)')::int,     
    substring(reloptions::text, 'autovacuum_freeze_table_age=(\d+)')::int     
  ),    
  a.v1    
)    
-    
age(case when relfrozenxid::text::int<3 then null else relfrozenxid end)     
as remain_ages_xid,   -- 再产生多少个事务后, 自动垃圾回收会触发FREEZE, 起因为事务ID    
coalesce(    
  least(    
    substring(reloptions::text, 'autovacuum_multixact_freeze_max_age=(\d+)')::int,     
    substring(reloptions::text, 'autovacuum_multixact_freeze_table_age=(\d+)')::int     
  ),    
  a.v2    
)    
-    
age(case when relminmxid::text::int<3 then null else relminmxid end)     
as remain_ages_mxid,  -- 再产生多少个事务后, 自动垃圾回收会触发FREEZE, 起因为并发事务ID    
coalesce(    
  least(    
    substring(reloptions::text, 'autovacuum_freeze_min_age=(\d+)')::int    
  ),    
  a.v3    
) as xid_lower_to_minage,    -- 如果触发FREEZE, 该表的事务ID年龄会降到多少    
coalesce(    
  least(    
    substring(reloptions::text, 'autovacuum_multixact_freeze_min_age=(\d+)')::int    
  ),    
  a.v4    
) as mxid_lower_to_minage,   -- 如果触发FREEZE, 该表的并行事务ID年龄会降到多少    
case     
  when v5 <= age(case when relfrozenxid::text::int<3 then null else relfrozenxid end) then 'YES'    
  else 'NOT'    
end as vacuum_trigger_freeze1,    -- 如果手工执行VACUUM, 是否会触发FREEZE, 触发起因(事务ID年龄达到阈值)    
case     
  when v6 <= age(case when relminmxid::text::int<3 then null else relminmxid end) then 'YES'    
  else 'NOT'    
end as vacuum_trigger_freeze2,    -- 如果手工执行VACUUM, 是否会触发FREEZE, 触发起因(并行事务ID年龄达到阈值)    
reloptions                        -- 表级参数, 优先. 例如是否开启自动垃圾回收, autovacuum_freeze_max_age, autovacuum_freeze_table_age, autovacuum_multixact_freeze_max_age, autovacuum_multixact_freeze_table_age    
from pg_class     
  where relkind in ('r','m')    
) e     
order by     
  least(e.remain_ages_xid , e.remain_ages_mxid),  -- 排在越前, 越先触发自动FREEZE, 即风暴来临的预测    
  pg_total_relation_size(reloid) desc   -- 同样剩余年龄, 表越大, 排越前    
;    

create view dba.v_freeze_stat as    
select     
wb,                                                     -- 第几个BATCH, 每个batch代表流逝100万个事务     
cnt,                                                    -- 这个batch 有多少表    
pg_size_pretty(ssz) as ssz1,                            -- 这个batch 这些 表+TOAST+索引 有多少容量    
pg_size_pretty(ssz) as ssz2,                            -- 这个batch FREEZE 会导致多少读IO    
pg_size_pretty(ssz*3) as ssz3,                          -- 这个batch FREEZE 最多可能会导致多少写IO (通常三份 : 数据文件, WAL FULL PAGE, WAL)    
pg_size_pretty(min_sz) as ssz4,                         -- 这个batch 最小的表多大    
pg_size_pretty(max_sz) as ssz5,                         -- 这个batch 最大的表多大    
pg_size_pretty(avg_sz) as ssz6,                         -- 这个batch 平均表多大    
pg_size_pretty(stddev_sz) as ssz7,                      -- 这个batch 表大小的方差, 越大, 说明表大小差异化明显    
min_rest_age,                                           -- 这个batch 距离自动FREEZE最低剩余事务数    
max_rest_age,                                           -- 这个batch 距离自动FREEZE最高剩余事务数    
stddev_rest_age,                                        -- 这个batch 距离自动FREEZE剩余事务数的方差, 越小，说明这个batch触发freeze将越平缓, 越大, 说明这个batch将有可能在某些点集中触发freeze (但是可能集中触发的都是小表)    
corr_rest_age_sz,                                       -- 表大小与距离自动freeze剩余事务数的相关性，相关性越强(值趋向1或-1) stddev_rest_age 与 sz7 说明的问题越有价值    
round(100*(ssz/(sum(ssz) over ())), 2)||' %' as ratio   -- 这个BATCH的容量占比，占比如果非常不均匀，说明有必要调整表级FREEZE参数，让占比均匀化    
from         
(    
select a.*, b.* from     
(    
select     
  min(least(remain_ages_xid, remain_ages_mxid)) as v_min,   -- 整个数据库中离自动FREEZE的 最小 剩余事务ID数    
  max(least(remain_ages_xid, remain_ages_mxid)) as v_max    -- 整个数据库中离自动FREEZE的 最大 剩余事务ID数    
from v_freeze    
) as a,    
LATERAL (  -- 高级SQL    
select     
width_bucket(    
  least(remain_ages_xid, remain_ages_mxid),     
  a.v_min,    
  a.v_max,    
  greatest((a.v_max-a.v_min)/1000000, 1)   -- 100万个事务, 如果要更改统计例如，修改这个值即可    
) as wb,      
count(*) as cnt,     
sum(pg_total_relation_size(reloid)) as ssz,     
stddev_samp(pg_total_relation_size(reloid) order by least(remain_ages_xid, remain_ages_mxid)) as stddev_sz,     
min(pg_total_relation_size(reloid)) as min_sz,     
max(pg_total_relation_size(reloid)) as max_sz,     
avg(pg_total_relation_size(reloid)) as avg_sz,     
min(least(remain_ages_xid, remain_ages_mxid)) as min_rest_age,     
max(least(remain_ages_xid, remain_ages_mxid)) as max_rest_age,     
stddev_samp(least(remain_ages_xid, remain_ages_mxid) order by least(remain_ages_xid, remain_ages_mxid)) as stddev_rest_age,     
corr(least(remain_ages_xid, remain_ages_mxid), pg_total_relation_size(reloid)) as corr_rest_age_sz     
from v_freeze     
group by wb     
) as b     
) t     
order by wb; 

create view dba.v_freeze_stat_detail as      
select     
pg_size_pretty(t.ssz) as ssz2,     -- 这个batch FREEZE 会导致多少读IO (表+TOAST+索引)    
pg_size_pretty(t.ssz*3) as ssz3,   -- 这个batch FREEZE 最多可能会导致多少写IO (通常三份 : 数据文件, WAL FULL PAGE, WAL)    
pg_size_pretty(t.ssz_sum) as ssz4, -- 所有batch 所有表的总大小  (表+TOAST+索引)    
round(100*(t.ssz/t.ssz_sum), 2)||' %' as ratio_batch,     -- 这个BATCH的容量占比，目标是让所有BATCH占比尽量一致    
round(100*(pg_total_relation_size(t.reloid)/t.ssz), 2)||' %' as ratio_table,     -- 这个表占整个batch的容量占比，大表尽量错开freeze    
t.*      
from         
(    
select a.*, b.* from       
(    
  select     
    min(least(remain_ages_xid, remain_ages_mxid)) as v_min,   -- 整个数据库中离自动FREEZE的 最小 剩余事务ID数    
    max(least(remain_ages_xid, remain_ages_mxid)) as v_max    -- 整个数据库中离自动FREEZE的 最大 剩余事务ID数    
  from v_freeze     
) as a,     
LATERAL (     -- 高级SQL    
select     
  count(*) over w as cnt,                                                -- 这个batch 有多少表      
  sum(pg_total_relation_size(reloid)) over () as ssz_sum,                -- 所有batch 所有表的总大小  (表+TOAST+索引)    
  sum(pg_total_relation_size(reloid)) over w as ssz,                     -- 这个batch 的表大小总和 (表+TOAST+索引)    
  pg_size_pretty(min(pg_total_relation_size(reloid)) over w) as min_sz,  -- 这个batch 最小的表多大    
  pg_size_pretty(max(pg_total_relation_size(reloid)) over w) as max_sz,  -- 这个batch 最大的表多大    
  pg_size_pretty(avg(pg_total_relation_size(reloid)) over w) as avg_sz,  -- 这个batch 平均表多大    
  pg_size_pretty(stddev_samp(pg_total_relation_size(reloid)) over w) as stddev_sz,  -- 这个batch 表大小的方差, 越大, 说明表大小差异化明显                                                                                                                 
  min(least(remain_ages_xid, remain_ages_mxid)) over w as min_rest_age,             -- 这个batch 距离自动FREEZE最低剩余事务数                                                                                                                             
  max(least(remain_ages_xid, remain_ages_mxid)) over w as max_rest_age,             -- 这个batch 距离自动FREEZE最高剩余事务数                                                                                                                             
  stddev_samp(least(remain_ages_xid, remain_ages_mxid)) over w as stddev_rest_age,  -- 这个batch 距离自动FREEZE剩余事务数的方差, 越小，说明这个batch触发freeze将越平缓, 越大, 说明这个batch将有可能在某些点集中触发freeze (但是可能集中触发的都是小表)    
  corr(least(remain_ages_xid, remain_ages_mxid), pg_total_relation_size(reloid)) over w as corr_rest_age_sz,  -- 表大小与距离自动freeze剩余事务数的相关性，相关性越强(值趋向1或-1) stddev_rest_age 与 stddev_sz 说明的问题越有价值    
  t1.*     
from     
  (    
  select     
    width_bucket(    
      least(tt.remain_ages_xid, tt.remain_ages_mxid),     
      a.v_min,    
      a.v_max,    
      greatest((a.v_max-a.v_min)/1000000, 1)         -- 100万个事务, 如果要更改统计例如，修改这个值即可    
    )     
    as wb,                                           -- 第几个BATCH, 每个batch代表流逝100万个事务      
    * from v_freeze tt    
  ) as t1      
  window w as     
  (    
    partition by t1.wb     
  )     
) as b    
) t    
order by     
  t.wb,      
  least(t.remain_ages_xid, t.remain_ages_mxid),       
  pg_total_relation_size(t.reloid) desc       
;      
  
create view dba.top20freezebigtable as 
select relowner::regrole, relnamespace::regnamespace, relname, 
age(relfrozenxid),pg_size_pretty(pg_total_relation_size(oid)) , -- 当前年龄 
coalesce(    
  least(    
    substring(reloptions::text, 'autovacuum_freeze_max_age=(\d+)')::int,     
    substring(reloptions::text, 'autovacuum_freeze_table_age=(\d+)')::int     
  ),    
  current_setting('autovacuum_freeze_max_age')::int   
)    
-    
age(case when relfrozenxid::text::int<3 then null else relfrozenxid end)     
as remain_ages_xid,  -- 再产生多少个事务后, 自动垃圾回收会触发FREEZE, 起因为事务ID
coalesce(    
  least(    
    substring(reloptions::text, 'autovacuum_freeze_min_age=(\d+)')::int    
  ),    
  current_setting('vacuum_freeze_min_age')::int   
) as xid_lower_to_minage    -- 如果触发FREEZE, 该表的事务ID年龄会降到多少  
from pg_class where relkind='r' order by pg_total_relation_size(oid) desc limit 20; 
```

# 未归档wal文件
```
create view dba.arch_undone as 
select * from pg_ls_archive_statusdir() where name !~ 'done$';
```
# 归档任务状态
```
create view dba.arch_status as
select * from pg_stat_get_archiver();
```
# wal空间占用
```
create view dba.walsize as 
select pg_size_pretty(sum(size)) from pg_ls_waldir();
```
# 系统强制保留wal大小
```
create view dba.wal_keep_size as
with a as (select setting from pg_settings where name='wal_keep_segments') , b as (select setting,unit from pg_settings where name='wal_segment_size') select pg_size_pretty(a.setting::int8*b.setting::int8) from a,b;
```
# 长事务、prepared statement
```
create view dba.long_snapshot as 
with a as (select min(transaction::Text::int8) m from pg_prepared_xacts ),
b as (select txid_snapshot_xmin(txid_current_snapshot())::text::int8 as m),
c as (select min(least(backend_xid::text::int8,backend_xmin::text::int8)) m from pg_stat_activity ),
d as (select datname,usename,pid,query_start,xact_start,now(),wait_event,query from pg_stat_activity where backend_xid is not null or backend_xmin is not null
order by least(backend_xid::text::int8,backend_xmin::text::int8) limit 1),
e as (select * from pg_prepared_xacts order by transaction::Text::int8 limit 1)
select b.m-least(a.m,c.m),d.*,e.* from a,b,c,d left join e on (1=1);
```

# 重置top query统计计数器(通常在高峰期来临前可以重置,防止结果干扰)
```
select pg_stat_statements_reset();
```
# 查询活跃会话数, 如果超过CPU核数, 说明数据库非常非常繁忙, 需要注意优化
```
create view dba.session_acting_cnt as select count(*) from pg_stat_activity where wait_event is not null and (backend_xid is not null or backend_xmin is not null); 
```

# 当前活跃会话
```
create view dba.sessions as select * from pg_stat_activity where wait_event is not null and (backend_xid is not null or backend_xmin is not null);  
```
# 查看锁等待
```
create view dba.locks as with      
t_wait as      
(      
  select a.mode,a.locktype,a.database,a.relation,a.page,a.tuple,a.classid,a.granted,     
  a.objid,a.objsubid,a.pid,a.virtualtransaction,a.virtualxid,a.transactionid,a.fastpath,      
  b.state,b.query,b.xact_start,b.query_start,b.usename,b.datname,b.client_addr,b.client_port,b.application_name     
    from pg_locks a,pg_stat_activity b where a.pid=b.pid and not a.granted     
),     
t_run as     
(     
  select a.mode,a.locktype,a.database,a.relation,a.page,a.tuple,a.classid,a.granted,     
  a.objid,a.objsubid,a.pid,a.virtualtransaction,a.virtualxid,a.transactionid,a.fastpath,     
  b.state,b.query,b.xact_start,b.query_start,b.usename,b.datname,b.client_addr,b.client_port,b.application_name     
    from pg_locks a,pg_stat_activity b where a.pid=b.pid and a.granted     
),     
t_overlap as     
(     
  select r.* from t_wait w join t_run r on     
  (     
    r.locktype is not distinct from w.locktype and     
    r.database is not distinct from w.database and     
    r.relation is not distinct from w.relation and     
    r.page is not distinct from w.page and     
    r.tuple is not distinct from w.tuple and     
    r.virtualxid is not distinct from w.virtualxid and     
    r.transactionid is not distinct from w.transactionid and     
    r.classid is not distinct from w.classid and     
    r.objid is not distinct from w.objid and     
    r.objsubid is not distinct from w.objsubid and     
    r.pid <> w.pid     
  )      
),      
t_unionall as      
(      
  select r.* from t_overlap r      
  union all      
  select w.* from t_wait w      
)      
select locktype,datname,relation::regclass,page,tuple,virtualxid,transactionid::text,classid::regclass,objid,objsubid,     
string_agg(     
'Pid: '||case when pid is null then 'NULL' else pid::text end||chr(10)||     
'Lock_Granted: '||case when granted is null then 'NULL' else granted::text end||' , Mode: '||case when mode is null then 'NULL' else mode::text end||' , FastPath: '||case when fastpath is null then 'NULL' else fastpath::text end||' , VirtualTransaction: '||case when virtualtransaction is null then 'NULL' else virtualtransaction::text end||' , Session_State: '||case when state is null then 'NULL' else state::text end||chr(10)||     
'Username: '||case when usename is null then 'NULL' else usename::text end||' , Database: '||case when datname is null then 'NULL' else datname::text end||' , Client_Addr: '||case when client_addr is null then 'NULL' else client_addr::text end||' , Client_Port: '||case when client_port is null then 'NULL' else client_port::text end||' , Application_Name: '||case when application_name is null then 'NULL' else application_name::text end||chr(10)||      
'Xact_Start: '||case when xact_start is null then 'NULL' else xact_start::text end||' , Query_Start: '||case when query_start is null then 'NULL' else query_start::text end||' , Xact_Elapse: '||case when (now()-xact_start) is null then 'NULL' else (now()-xact_start)::text end||' , Query_Elapse: '||case when (now()-query_start) is null then 'NULL' else (now()-query_start)::text end||chr(10)||      
'SQL (Current SQL in Transaction): '||chr(10)||    
case when query is null then 'NULL' else query::text end,      
chr(10)||'--------'||chr(10)      
order by      
  (  case mode      
    when 'INVALID' then 0     
    when 'AccessShareLock' then 1     
    when 'RowShareLock' then 2     
    when 'RowExclusiveLock' then 3     
    when 'ShareUpdateExclusiveLock' then 4     
    when 'ShareLock' then 5     
    when 'ShareRowExclusiveLock' then 6     
    when 'ExclusiveLock' then 7     
    when 'AccessExclusiveLock' then 8     
    else 0     
  end  ) desc,     
  (case when granted then 0 else 1 end)    
) as lock_conflict    
from t_unionall     
group by     
locktype,datname,relation,page,tuple,virtualxid,transactionid::text,classid,objid,objsubid ;
```

# 查看索引支持类型
```
SELECT
    am.amname AS index_method,
    opc.opcname AS opclass_name,
    opc.opcintype::regtype AS indexed_type,
    opc.opcdefault AS is_default
FROM
    pg_am am,
    pg_opclass opc
WHERE
    opc.opcmethod = am.oid
ORDER BY
    index_method,
    opclass_name;
SELECT
    amname,
    opcname,
    opcintype::regtype,
    opckeytype::regtype,
    opcdefault
FROM
    pg_am am,
    pg_opclass opc
WHERE
    am.oid = opc.opcmethod
    AND amname = 'gin';

SELECT
    oid,
    oprnegate,
    oprname,
    oprcode,
    oprresult::regtype,
    oprleft::regtype,
    oprright::regtype,
    oprcanmerge
FROM
    pg_operator;

```
# 判断ip是否在某一网段内
```
CREATE OR REPLACE FUNCTION public.is_same_network (ip1 ip4, ip2 ip4, mask integer)
    RETURNS boolean
AS
$$
DECLARE
    is_same_network boolean;
BEGIN
    IF mask > 32 OR mask < 0 THEN
        raise
        exception 'The mask must be between 0 and 32';
    END IF;
 
    EXECUTE format('select (~($1 # $2))::bigint::bit(32)::bit(%I)::text ~ ''^1+$''', mask) using ip1, ip2 into is_same_network;
    RETURN is_same_network;
exception
    WHEN OTHERS THEN
        raise NOTICE '%', SQLERRM;
    RETURN FALSE;
END;
$$ language plpgsql;
```
# 指定字符替换
```
postgres=# SELECT regexp_replace('foobarbaz', 'b(..)', 'X\1Y', 'g');
 regexp_replace 
----------------
 fooXarYXazY
(1 row)

```
#  SQL实现圣诞树
```
mydb=# WITH leaf AS
 (SELECT lpad(rpad('*', (id - 1) * 2 + 1, '*'), id + 20) leaf,
         id
    FROM generate_series(1, 3) AS t(id)),
lv AS
 (SELECT id lv FROM generate_series(1, 5) AS t(id)),
leafs AS
 (SELECT lpad(rpad('*', ((row_number() over()) ::INT - 1) * 2 + 1 + (lv - 1) * 2, '*'), (row_number()
                over())
               ::INT + 20 + lv) leaf
    FROM leaf,
         lv),
root AS
 (SELECT lpad(rpad('*', 5, '*'), 24) FROM generate_series(1, 4) AS t(id))
SELECT leaf
  FROM leafs
UNION ALL
SELECT * FROM root;
                   leaf                  
------------------------------------------
                      *
                    *****
                  *********
                *************
              *****************
                 ***********
               ***************
             *******************
           ***********************
         ***************************
            *********************
          *************************
        *****************************
      *********************************
    *************************************
                    *****
                    *****
                    *****
                    *****
(19 rows)
 
mydb=#

```
#  生成随机中文

```sql
create or replace function gen_hanzi(int) returns text as $$  
declare  
  res text;  
begin  
  if $1 >=1 then  
    select string_agg(chr(19968+(random()*20901)::int), '') into res from generate_series(1,$1);  
    return res;  
  end if;  
  return null;  
end;  
$$ language plpgsql strict;
```
#  生成随机时间
```
create or replace function get_rand_ts() returns timestamp as $$  
  select now()::timestamp  +  ((1000*random())::int::text||' days')::interval;            
$$ language sql strict;  
```
# 找出index 维护SQL
```sql
select
        CASE
                WHEN flag = 1 THEN
                        CASE
                        WHEN indexdef !~ ' WHERE ' THEN
                                regexp_replace(indexdef, E'(INDEX )(.+)( ON )(.+\\)\$)'  ,E' \\1 CONCURRENTLY \\3 \\4 TABLESPACE  pg_default ','g') ||'; '
                        ELSE
                                regexp_replace(indexdef, E'(INDEX )(.+)( ON )(.+)( WHERE )'  ,E' \\1 CONCURRENTLY \\3 \\4 TABLESPACE  pg_default \\5 ','g') ||'; '
                        END
                WHEN flag = 2 THEN
                                'ANALYZE VERBOSE '||schemaname||'.'||tablename||' ; select pg_sleep(600) ; DROP INDEX CONCURRENTLY IF EXISTS '||schemaname||'.'||indexname||'; '
        END
from
        (
        select
                generate_series(1,2) as flag,
                indexdef,
                indexname,
                tablename,
                pi.schemaname
        from
                pg_indexes pi
        join
                pg_namespace n
          on
                pi.schemaname = n.nspname
        join
                pg_class pcl
          on
                pcl.relnamespace = n.oid
                and pcl.relname = pi.tablename
        left join
                pg_constraint pco
          on
                pco.conname = pi.indexname
                and pco.conrelid = pcl.oid
        where
                (pi.schemaname, pi.tablename) = ('mirror','b2c_order')
                and pco.contype is distinct from  'p'
                and pco.contype is distinct from  'u'
        order by
                pi.schemaname, tablename, indexname, pg_table_size(schemaname||'.'||indexname::text) desc, flag asc
        ) as foo
order by
       schemaname, tablename, indexname, pg_table_size(schemaname||'.'||indexname::text) desc, flag asc
;
```

# 参考
[德哥](https://github.com/digoal/blog/blob/master/202005/20200509_02.md)
