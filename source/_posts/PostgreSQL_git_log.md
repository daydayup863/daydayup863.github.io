---
title: PostgreSQL git commit 阅读记录
date: 2021-01-22 11:43:14
tags: 
- PostgreSQL
- git 
- commit log
categories: 
- PostgreSQL
top: 30
description: 
password: 

---


记录阅读PostgreSQL git 提交记录, 觉得好的记录一下.

<!-- more -->

# PG14

## Support tab-complete for TRUNCATE on foreign tables.
```
commit 81e094bdfdd6cf6568cba2b25eea9876daceaacb
Author: Fujii Masao <fujii@postgresql.org>
Date:   Mon Apr 12 21:34:23 2021 +0900

    Support tab-complete for TRUNCATE on foreign tables.

    Commit 8ff1c94649 extended TRUNCATE command so that it can also truncate
    foreign tables. But it forgot to support tab-complete for TRUNCATE on
    foreign tables. That is, previously tab-complete for TRUNCATE displayed
    only the names of regular tables.

    This commit improves tab-complete for TRUNCATE so that it displays also
    the names of foreign tables.

    Author: Fujii Masao
    Reviewed-by: Bharath Rupireddy
    Discussion: https://postgr.es/m/551ed8c1-f531-818b-664a-2cecdab99cd8@oss.nttdata.com

```

## Add information of total data processed to replication slot stats.
```
commit f5fc2f5b23d1b1dff60f8ca5dc211161df47eda4 (HEAD -> master, origin/master, origin/HEAD)
Author: Amit Kapila <akapila@postgresql.org>
Date:   Fri Apr 16 07:34:43 2021 +0530

    Add information of total data processed to replication slot stats.

    This adds the statistics about total transactions count and total
    transaction data logically sent to the decoding output plugin from
    ReorderBuffer. Users can query the pg_stat_replication_slots view to check
    these stats.

    Suggested-by: Andres Freund
    Author: Vignesh C and Amit Kapila
    Reviewed-by: Sawada Masahiko, Amit Kapila
    Discussion: https://postgr.es/m/20210319185247.ldebgpdaxsowiflw@alap3.anarazel.de


```

## Add missing COMPRESSION into CREATE TABLE synopsis.
```
commit e2e2efca85b4857361780ed0c736c2a44edb458a
Author: Fujii Masao <fujii@postgresql.org>
Date:   Thu Apr 15 23:15:19 2021 +0900

    doc: Add missing COMPRESSION into CREATE TABLE synopsis.

    Commit bbe0a81db6 introduced "INCLUDING COMPRESSION" option
    in CREATE TABLE command, but forgot to mention it in the
    CREATE TABLE syntax synopsis.

    Author: Fujii Masao
    Reviewed-by: Michael Paquier
    Discussion: https://postgr.es/m/54d30e66-dbd6-5485-aaf6-a291ed55919d@oss.nttdata.com

```



## 新增语法ALTER SUBSCRIPTION ... ADD/DROP PUBLICATION
```
commit 82ed7748b710e3ddce3f7ebc74af80fe4869492f
Author: Peter Eisentraut <peter@eisentraut.org>
Date:   Tue Apr 6 10:44:26 2021 +0200

    ALTER SUBSCRIPTION ... ADD/DROP PUBLICATION

    At present, if we want to update publications in a subscription, we
    can use SET PUBLICATION.  However, it requires supplying all
    publications that exists and the new publications.  If we want to add
    new publications, it's inconvenient.  The new syntax only supplies the
    new publications.  When the refresh is true, it only refreshes the new
    publications.

    Author: Japin Li <japinli@hotmail.com>
    Author: Bharath Rupireddy <bharath.rupireddyforpostgres@gmail.com>
    Discussion: https://www.postgresql.org/message-id/flat/MEYP282MB166939D0D6C480B7FBE7EFFBB6BC0@MEYP282MB1669.AUSP282.PROD.OUTLOOK.COM

```

## SP-GiST. Support INCLUDE'd columns
```
commit 09c1c6ab4bc5764dd69c53ccfd43b2060b1fd090
Author: Tom Lane <tgl@sss.pgh.pa.us>
Date:   Mon Apr 5 18:41:09 2021 -0400

    Support INCLUDE'd columns in SP-GiST.

    Not much to say here: does what it says on the tin.
    We steal a previously-always-zero bit from the nextOffset
    field of leaf index tuples in order to track whether there
    is a nulls bitmap.  Otherwise it works about like included
    columns in other index types.

    Pavel Borisov, reviewed by Andrey Borodin and Anastasia Lubennikova,
    and rather heavily editorialized on by me

    Discussion: https://postgr.es/m/CALT9ZEFi-vMp4faht9f9Junb1nO3NOSjhpxTmbm1UGLMsLqiEQ@mail.gmail.com

```

## Change return type of EXTRACT from float8 to numeric
```
commit a2da77cdb4661826482ebf2ddba1f953bc74afe4
Author: Peter Eisentraut <peter@eisentraut.org>
Date:   Tue Apr 6 07:17:13 2021 +0200

    Change return type of EXTRACT to numeric

    The previous implementation of EXTRACT mapped internally to
    date_part(), which returned type double precision (since it was
    implemented long before the numeric type existed).  This can lead to
    imprecise output in some cases, so returning numeric would be
    preferrable.  Changing the return type of an existing function is a
    bit risky, so instead we do the following:  We implement a new set of
    functions, which are now called "extract", in parallel to the existing
    date_part functions.  They work the same way internally but use
    numeric instead of float8.  The EXTRACT construct is now mapped by the
    parser to these new extract functions.  That way, dumps of views
    etc. from old versions (which would use date_part) continue to work
    unchanged, but new uses will map to the new extract functions.

    Additionally, the reverse compilation of EXTRACT now reproduces the
    original syntax, using the new mechanism introduced in
    40c24bfef92530bd846e111c1742c2a54441c62c.

    The following minor changes of behavior result from the new
    implementation:

    - The column name from an isolated EXTRACT call is now "extract"
      instead of "date_part".

    - Extract from date now rejects inappropriate field names such as
      HOUR.  It was previously mapped internally to extract from
      timestamp, so it would silently accept everything appropriate for
      timestamp.

    - Return values when extracting fields with possibly fractional
      values, such as second and epoch, now have the full scale that the
      value has internally (so, for example, '1.000000' instead of just
      '1').

    Reported-by: Petr Fedorov <petr.fedorov@phystech.edu>
    Reviewed-by: Tom Lane <tgl@sss.pgh.pa.us>
    Discussion: https://www.postgresql.org/message-id/flat/42b73d2d-da12-ba9f-570a-420e0cce19d9@phystech.edu

```

## Add function pg_log_backend_memory_contexts to log the memory contexts of specified backend process.
```
commit 43620e328617c1f41a2a54c8cee01723064e3ffa
Author: Fujii Masao <fujii@postgresql.org>
Date:   Tue Apr 6 13:44:15 2021 +0900

    Add function to log the memory contexts of specified backend process.

    Commit 3e98c0bafb added pg_backend_memory_contexts view to display
    the memory contexts of the backend process. However its target process
    is limited to the backend that is accessing to the view. So this is
    not so convenient when investigating the local memory bloat of other
    backend process. To improve this situation, this commit adds
    pg_log_backend_memory_contexts() function that requests to log
    the memory contexts of the specified backend process.

    This information can be also collected by calling
    MemoryContextStats(TopMemoryContext) via a debugger. But
    this technique cannot be used in some environments because no debugger
    is available there. So, pg_log_backend_memory_contexts() allows us to
    see the memory contexts of specified backend more easily.

    Only superusers are allowed to request to log the memory contexts
    because allowing any users to issue this request at an unbounded rate
    would cause lots of log messages and which can lead to denial of service.

    On receipt of the request, at the next CHECK_FOR_INTERRUPTS(),
    the target backend logs its memory contexts at LOG_SERVER_ONLY level,
    so that these memory contexts will appear in the server log but not
    be sent to the client. It logs one message per memory context.
    Because if it buffers all memory contexts into StringInfo to log them
    as one message, which may require the buffer to be enlarged very much
    and lead to OOM error since there can be a large number of memory
    contexts in a backend.

    When a backend process is consuming huge memory, logging all its
    memory contexts might overrun available disk space. To prevent this,
    now this patch limits the number of child contexts to log per parent
    to 100. As with MemoryContextStats(), it supposes that practical cases
    where the log gets long will typically be huge numbers of siblings
    under the same parent context; while the additional debugging value
    from seeing details about individual siblings beyond 100 will not be large.

    There was another proposed patch to add the function to return
    the memory contexts of specified backend as the result sets,
    instead of logging them, in the discussion. However that patch is
    not included in this commit because it had several issues to address.

    Thanks to Tatsuhito Kasahara, Andres Freund, Tom Lane, Tomas Vondra,
    Michael Paquier, Kyotaro Horiguchi and Zhihong Yu for the discussion.

    Bump catalog version.

    Author: Atsushi Torikoshi
    Reviewed-by: Kyotaro Horiguchi, Zhihong Yu, Fujii Masao
    Discussion: https://postgr.es/m/0271f440ac77f2a4180e0e56ebd944d1@oss.nttdata.com

```
## new GUC recovery_prefetch
```
commit 1d257577e08d3e598011d6850fd1025858de8c8c
Author: Thomas Munro <tmunro@postgresql.org>
Date:   Thu Apr 8 23:03:43 2021 +1200

    Optionally prefetch referenced data in recovery.
    
    Introduce a new GUC recovery_prefetch, disabled by default.  When
    enabled, look ahead in the WAL and try to initiate asynchronous reading
    of referenced data blocks that are not yet cached in our buffer pool.
    For now, this is done with posix_fadvise(), which has several caveats.
    Better mechanisms will follow in later work on the I/O subsystem.
    
    The GUC maintenance_io_concurrency is used to limit the number of
    concurrent I/Os we allow ourselves to initiate, based on pessimistic
    heuristics used to infer that I/Os have begun and completed.
    
    The GUC wal_decode_buffer_size is used to limit the maximum distance we
    are prepared to read ahead in the WAL to find uncached blocks.
    
    Reviewed-by: Alvaro Herrera <alvherre@2ndquadrant.com> (parts)
    Reviewed-by: Andres Freund <andres@anarazel.de> (parts)
    Reviewed-by: Tomas Vondra <tomas.vondra@2ndquadrant.com> (parts)
    Tested-by: Tomas Vondra <tomas.vondra@2ndquadrant.com>
    Tested-by: Jakub Wartak <Jakub.Wartak@tomtom.com>
    Tested-by: Dmitry Dolgov <9erthalion6@gmail.com>
    Tested-by: Sait Talha Nisanci <Sait.Nisanci@microsoft.com>
    Discussion: https://postgr.es/m/CA%2BhUKGJ4VJN8ttxScUFM8dOKX0BrBiboo5uz1cq%3DAovOddfHpA%40mail.gmail.com

```

## new GUC wal_decode_buffer_size.
```
commit f003d9f8721b3249e4aec8a1946034579d40d42c
Author: Thomas Munro <tmunro@postgresql.org>
Date:   Thu Apr 8 23:03:34 2021 +1200

    Add circular WAL decoding buffer.

    Teach xlogreader.c to decode its output into a circular buffer, to
    support optimizations based on looking ahead.

     * XLogReadRecord() works as before, consuming records one by one, and
       allowing them to be examined via the traditional XLogRecGetXXX()
       macros.

     * An alternative new interface XLogNextRecord() is added that returns
       pointers to DecodedXLogRecord structs that can be examined directly.

     * XLogReadAhead() provides a second cursor that lets you see
       further ahead, as long as data is available and there is enough space
       in the decoding buffer.  This returns DecodedXLogRecord pointers to the
       caller, but also adds them to a queue of records that will later be
       consumed by XLogNextRecord()/XLogReadRecord().

    The buffer's size is controlled with wal_decode_buffer_size.  The buffer
    could potentially be placed into shared memory, for future projects.
    Large records that don't fit in the circular buffer are called
    "oversized" and allocated separately with palloc().

    Discussion: https://postgr.es/m/CA+hUKGJ4VJN8ttxScUFM8dOKX0BrBiboo5uz1cq=AovOddfHpA@mail.gmail.com

```

## Add functions to wait for backend termination
```
commit aaf043257205ec523f1ba09a3856464d17cf2281
Author: Magnus Hagander <magnus@hagander.net>
Date:   Thu Apr 8 11:32:14 2021 +0200

    Add functions to wait for backend termination

    This adds a function, pg_wait_for_backend_termination(), and a new
    timeout argument to pg_terminate_backend(), which will wait for the
    backend to actually terminate (with or without signaling it to do so
    depending on which function is called). The default behaviour of
    pg_terminate_backend() remains being timeout=0 which does not waiting.
    For pg_wait_for_backend_termination() the default wait is 5 seconds.

    Author: Bharath Rupireddy
    Reviewed-By: Fujii Masao, David Johnston, Muhammad Usama,
                 Hou Zhijie, Magnus Hagander
    Discussion: https://postgr.es/m/CALj2ACUBpunmyhYZw-kXCYs5NM+h6oG_7Df_Tn4mLmmUQifkqA@mail.gmail.com

```

##  Add csvlog output for the new query_id value
```
commit f57a2f5e03054ade221e554c70e628e1ffae1b66
Author: Bruce Momjian <bruce@momjian.us>
Date:   Wed Apr 7 22:30:30 2021 -0400

    Add csvlog output for the new query_id value

    This also adjusts the printf format for query id used by log_line_prefix
    (%Q).

    Reported-by: Justin Pryzby

    Discussion: https://postgr.es/m/20210408005402.GG24239@momjian.us

    Author: Julien Rouhaud, Bruce Momjian

```

## SQL-standard function body
```
commit e717a9a18b2e34c9c40e5259ad4d31cd7e420750
Author: Peter Eisentraut <peter@eisentraut.org>
Date:   Wed Apr 7 21:30:08 2021 +0200

    SQL-standard function body

    This adds support for writing CREATE FUNCTION and CREATE PROCEDURE
    statements for language SQL with a function body that conforms to the
    SQL standard and is portable to other implementations.

    Instead of the PostgreSQL-specific AS $$ string literal $$ syntax,
    this allows writing out the SQL statements making up the body
    unquoted, either as a single statement:

        CREATE FUNCTION add(a integer, b integer) RETURNS integer
            LANGUAGE SQL
            RETURN a + b;

    or as a block

        CREATE PROCEDURE insert_data(a integer, b integer)
        LANGUAGE SQL
        BEGIN ATOMIC
          INSERT INTO tbl VALUES (a);
          INSERT INTO tbl VALUES (b);
        END;

    The function body is parsed at function definition time and stored as
    expression nodes in a new pg_proc column prosqlbody.  So at run time,
    no further parsing is required.

    However, this form does not support polymorphic arguments, because
    there is no more parse analysis done at call time.

    Dependencies between the function and the objects it uses are fully
    tracked.

    A new RETURN statement is introduced.  This can only be used inside
    function bodies.  Internally, it is treated much like a SELECT
    statement.

    psql needs some new intelligence to keep track of function body
    boundaries so that it doesn't send off statements when it sees
    semicolons that are inside a function body.

    Tested-by: Jaime Casanova <jcasanov@systemguards.com.ec>
    Reviewed-by: Julien Rouhaud <rjuju123@gmail.com>
    Discussion: https://www.postgresql.org/message-id/flat/1c11f1eb-f00c-43b7-799d-2d44132c02d7@2ndquadrant.com

```

## Enable parallel SELECT for "INSERT INTO ... SELECT ..."

```
commit 05c8482f7f69a954fd65fce85f896e848fc48197
Author: Amit Kapila <akapila@postgresql.org>
Date:   Wed Mar 10 07:38:58 2021 +0530

    Enable parallel SELECT for "INSERT INTO ... SELECT ...".

    Parallel SELECT can't be utilized for INSERT in the following cases:
    - INSERT statement uses the ON CONFLICT DO UPDATE clause
    - Target table has a parallel-unsafe: trigger, index expression or
      predicate, column default expression or check constraint
    - Target table has a parallel-unsafe domain constraint on any column
    - Target table is a partitioned table with a parallel-unsafe partition key
      expression or support function

    The planner is updated to perform additional parallel-safety checks for
    the cases listed above, for determining whether it is safe to run INSERT
    in parallel-mode with an underlying parallel SELECT. The planner will
    consider using parallel SELECT for "INSERT INTO ... SELECT ...", provided
    nothing unsafe is found from the additional parallel-safety checks, or
    from the existing parallel-safety checks for SELECT.

    While checking parallel-safety, we need to check it for all the partitions
    on the table which can be costly especially when we decide not to use a
    parallel plan. So, in a separate patch, we will introduce a GUC and or a
    reloption to enable/disable parallelism for Insert statements.

    Prior to entering parallel-mode for the execution of INSERT with parallel
    SELECT, a TransactionId is acquired and assigned to the current
    transaction state. This is necessary to prevent the INSERT from attempting
    to assign the TransactionId whilst in parallel-mode, which is not allowed.
    This approach has a disadvantage in that if the underlying SELECT does not
    return any rows, then the TransactionId is not used, however that
    shouldn't happen in practice in many cases.

    Author: Greg Nancarrow, Amit Langote, Amit Kapila
    Reviewed-by: Amit Langote, Hou Zhijie, Takayuki Tsunakawa, Antonin Houska, Bharath Rupireddy, Dilip Kumar, Vignesh C, Zhihong Yu, Amit Kapila
    Tested-by: Tang, Haiying
    Discussion: https://postgr.es/m/CAJcOf-cXnB5cnMKqWEp2E2z7Mvcd04iLVmV=qpFJrR3AcrTS3g@mail.gmail.com
    Discussion: https://postgr.es/m/CAJcOf-fAdj=nDKMsRhQzndm-O13NY4dL6xGcEvdX5Xvbbi0V7g@mail.gmail.com

commit 0ba71107efeeccde9158f47118f95043afdca0bb

```

## Add --tablespace option to reindexd
```
    Add --tablespace option to reindexdb

    This option provides REINDEX (TABLESPACE) for reindexdb, applying the
    tablespace value given by the caller to all the REINDEX queries
    generated.

    While on it, this commit adds some tests for REINDEX TABLESPACE, with
    and without CONCURRENTLY, when run on toast indexes and tables.  Such
    operations are not allowed, and toast relation names are not stable
    enough to be part of the main regression test suite (even if using a PL
    function with a TRY/CATCH logic, as CONCURRENTLY could not be tested).

    Author: Michael Paquier

```

## target_session_attrs 增加　read-only primary standby prefer-standby 支持
```
    Extend the abilities of libpq's target_session_attrs parameter.

    In addition to the existing options of "any" and "read-write", we
    now support "read-only", "primary", "standby", and "prefer-standby".
    "read-write" retains its previous meaning of "transactions are
    read-write by default", and "read-only" inverts that.  The other
    three modes test specifically for hot-standby status, which is not
    quite the same thing.  (Setting default_transaction_read_only on
    a primary server renders it read-only to this logic, but not a
    standby.)

    Furthermore, if talking to a v14 or later server, no extra network
    round trip is needed to detect the session's status; the GUC_REPORT
    variables delivered by the server are enough.  When talking to an
    older server, a SHOW or SELECT query is issued to detect session
    read-only-ness or server hot-standby state, as needed.

    Haribabu Kommi, Greg Nancarrow, Vignesh C, Tom Lane; reviewed at
    various times by Laurenz Albe, Takayuki Tsunakawa, Peter Smith.

    Discussion: https://postgr.es/m/CAF3+xM+8-ztOkaV9gHiJ3wfgENTq97QcjXQt+rbFQ6F7oNzt9A@mail.gmail.com

```

## Add option PROCESS_TOAST to VACUUM
```
Author: Michael Paquier <michael@paquier.xyz>
Date:   Tue Feb 9 14:13:57 2021 +0900

    Add option PROCESS_TOAST to VACUUM

    This option controls if toast tables associated with a relation are
    vacuumed or not when running a manual VACUUM.  It was already possible
    to trigger a manual VACUUM on a toast relation without processing its
    main relation, but a manual vacuum on a main relation always forced a
    vacuum on its toast table.  This is useful in scenarios where the level
    of bloat or transaction age of the main and toast relations differs a
    lot.

    This option is an extension of the existing VACOPT_SKIPTOAST that was
    used by autovacuum to control if toast relations should be skipped or
    not.  This internal flag is renamed to VACOPT_PROCESS_TOAST for
    consistency with the new option.

    A new option switch, called --no-process-toast, is added to vacuumdb.

    Author: Nathan Bossart
    Reviewed-by: Kirk Jamison, Michael Paquier, Justin Pryzby
    Discussion: https://postgr.es/m/BA8951E9-1524-48C5-94AF-73B1F0D7857F@amazon.com

```

## added "hostgssenc" type HBA entries
```
commit 3995c424984e991b1069a2869af972dc07574c0b
Author: Tom Lane <tgl@sss.pgh.pa.us>
Date:   Mon Dec 28 17:58:58 2020 -0500

    Improve log messages related to pg_hba.conf not matching a connection.

    Include details on whether GSS encryption has been activated;
    since we added "hostgssenc" type HBA entries, that's relevant info.

    Kyotaro Horiguchi and Tom Lane.  Back-patch to v12 where
    GSS encryption was introduced.

    Discussion: https://postgr.es/m/e5b0b6ed05764324a2f3fe7acfc766d5@smhi.se

```

## Introduce a new GUC_REPORT setting "in_hot_standby"
```
commit bf8a662c9afad6fd07b42cdc5e71416c51f75d31
Author: Tom Lane <tgl@sss.pgh.pa.us>
Date:   Tue Jan 5 16:18:01 2021 -0500

    Introduce a new GUC_REPORT setting "in_hot_standby".

    Aside from being queriable via SHOW, this value is sent to the client
    immediately at session startup, and again later on if the server gets
    promoted to primary during the session.  The immediate report will be
    used in an upcoming patch to avoid an extra round trip when trying to
    connect to a primary server.

    Haribabu Kommi, Greg Nancarrow, Tom Lane; reviewed at various times
    by Laurenz Albe, Takayuki Tsunakawa, Peter Smith.

    Discussion: https://postgr.es/m/CAF3+xM+8-ztOkaV9gHiJ3wfgENTq97QcjXQt+rbFQ6F7oNzt9A@mail.gmail.com

```

## Add idle_session_timeout　GUC 好东西, 不听话的应用连接就得杀
```
commit 9877374bef76ef03923f6aa8b955f2dbcbe6c2c7
Author: Tom Lane <tgl@sss.pgh.pa.us>
Date:   Wed Jan 6 18:28:42 2021 -0500

    Add idle_session_timeout.

    This GUC variable works much like idle_in_transaction_session_timeout,
    in that it kills sessions that have waited too long for a new client
    query.  But it applies when we're not in a transaction, rather than
    when we are.

    Li Japin, reviewed by David Johnston and Hayato Kuroda, some
    fixes by me

    Discussion: https://postgr.es/m/763A0689-F189-459E-946F-F0EC4458980B@hotmail.com


```

## Add GUC to log long wait times on recovery conflicts. 好东西，一定要开哟
```
commit 0650ff23038bc3eb8d8fd851744db837d921e285
Author: Fujii Masao <fujii@postgresql.org>
Date:   Fri Jan 8 00:47:03 2021 +0900

    Add GUC to log long wait times on recovery conflicts.

    This commit adds GUC log_recovery_conflict_waits that controls whether
    a log message is produced when the startup process is waiting longer than
    deadlock_timeout for recovery conflicts. This is useful in determining
    if recovery conflicts prevent the recovery from applying WAL.

    Note that currently a log message is produced only when recovery conflict
    has not been resolved yet even after deadlock_timeout passes, i.e.,
    only when the startup process is still waiting for recovery conflict
    even after deadlock_timeout.

    Author: Bertrand Drouvot, Masahiko Sawada
    Reviewed-by: Alvaro Herrera, Kyotaro Horiguchi, Fujii Masao
    Discussion: https://postgr.es/m/9a60178c-a853-1440-2cdc-c3af916cff59@amazon.com

```

## Add pg_stat_database counters for sessions and session time
```
commit 960869da0803427d14335bba24393f414b476e2c
Author: Magnus Hagander <magnus@hagander.net>
Date:   Sun Jan 17 13:34:09 2021 +0100

    Add pg_stat_database counters for sessions and session time

    This add counters for number of sessions, the different kind of session
    termination types, and timers for how much time is spent in active vs
    idle in a database to pg_stat_database.

    Internally this also renames the parameter "force" to disconnect. This
    was the only use-case for the parameter before, so repurposing it to
    this mroe narrow usecase makes things cleaner than inventing something
    new.

    Author: Laurenz Albe
    Reviewed-By: Magnus Hagander, Soumyadeep Chakraborty, Masahiro Ikeda
    Discussion: https://postgr.es/m/b07e1f9953701b90c66ed368656f2aef40cac4fb.camel@cybertec.at

```

## psql \dX: list extended statistics objects
```
commit 891a1d0bca262ca78564e0fea1eaa5ae544ea5ee
Author: Tomas Vondra <tomas.vondra@postgresql.org>
Date:   Sun Jan 17 00:16:25 2021 +0100

    psql \dX: list extended statistics objects

    The new command lists extended statistics objects, possibly with their
    sizes. All past releases with extended statistics are supported.

    Author: Tatsuro Yamada
    Reviewed-by: Julien Rouhaud, Alvaro Herrera, Tomas Vondra
    Discussion: https://postgr.es/m/c027a541-5856-75a5-0868-341301e1624b%40nttcom.co.jp_1
```
```
commit 1db0d173a2201119f297ea35edfb41579893dd8c
Author: Tomas Vondra <tomas.vondra@postgresql.org>
Date:   Sun Jan 17 15:11:14 2021 +0100

    Revert "psql \dX: list extended statistics objects"

    Reverts 891a1d0bca, because the new  psql command \dX only worked for
    users users who can read pg_statistic_ext_data catalog, and most regular
    users lack that privilege (the catalog may contain sensitive user data).

    Reported-by: Noriyoshi Shinoda
    Discussion: https://postgr.es/m/c027a541-5856-75a5-0868-341301e1624b%40nttcom.co.jp_1
```

## 感觉14版本做了很多FDW的优化，是为集群模式做准备?

```
commit 8e396a773b80c72e5d5a0ca9755dffe043c97a05
Author: Tom Lane <tgl@sss.pgh.pa.us>
Date:   Thu Jan 14 16:19:38 2021 -0500

    pg_dump: label PUBLICATION TABLE ArchiveEntries with an owner.

    This is the same fix as commit 9eabfe300 applied to INDEX ATTACH
    entries, but for table-to-publication attachments.  As in that
    case, even though the backend doesn't record "ownership" of the
    attachment, we still ought to label it in the dump archive with
    the role name that should run the ALTER PUBLICATION command.
    The existing behavior causes the ALTER to be done by the original
    role that started the restore; that will usually work fine, but
    there may be corner cases where it fails.

    The bulk of the patch is concerned with changing struct
    PublicationRelInfo to include a pointer to the associated
    PublicationInfo object, so that we can get the owner's name
    out of that when the time comes.  While at it, I rewrote
    getPublicationTables() to do just one query of pg_publication_rel,
    not one per table.

    Back-patch to v10 where this code was introduced.

    Discussion: https://postgr.es/m/1165710.1610473242@sss.pgh.pa.us
```

```
commit 708d165ddb92c54077a372acf6417e258dcb5fef
Author: Fujii Masao <fujii@postgresql.org>
Date:   Mon Jan 18 15:11:08 2021 +0900

    postgres_fdw: Add function to list cached connections to foreign servers.

    This commit adds function postgres_fdw_get_connections() to return
    the foreign server names of all the open connections that postgres_fdw
    established from the local session to the foreign servers. This function
    also returns whether each connection is valid or not.

    This function is useful when checking all the open foreign server connections.
    If we found some connection to drop, from the result of function, probably
    we can explicitly close them by the function that upcoming commit will add.

    This commit bumps the version of postgres_fdw to 1.1 since it adds
    new function.

    Author: Bharath Rupireddy, tweaked by Fujii Masao
    Reviewed-by: Zhijie Hou, Alexey Kondratov, Zhihong Yu, Fujii Masao
    Discussion: https://postgr.es/m/2d5cb0b3-a6e8-9bbb-953f-879f47128faa@oss.nttdata.com

```

```
commit ee79a548e746da9a99df0cac70a3ddc095f2829a
Author: Fujii Masao <fujii@postgresql.org>
Date:   Tue Jan 19 00:56:10 2021 +0900

    doc: Add note about the server name of postgres_fdw_get_connections() returns.

    Previously the document didn't mention the case where
    postgres_fdw_get_connections() returns NULL in server_name column.
    Users might be confused about why NULL was returned.

    This commit adds the note that, in postgres_fdw_get_connections(),
    the server name of an invalid connection will be NULL if the server is dropped.

    Suggested-by: Zhijie Hou
    Author: Bharath Rupireddy
    Reviewed-by: Zhijie Hou, Fujii Masao
    Discussion: https://postgr.es/m/e7ddd14e96444fce88e47a709c196537@G08CNEXMBPEKD05.g08.fujitsu.local
```

```
commit e3ebcca843a4703938b9f40a4811f43c4b315753
Author: Fujii Masao <fujii@postgresql.org>
Date:   Mon Dec 28 19:56:13 2020 +0900

    postgres_fdw: Fix connection leak.

    In postgres_fdw, the cached connections to foreign servers will not be
    closed until the local session exits if the user mappings or foreign servers
    that those connections depend on are dropped. Those connections can be
    leaked.

    To fix that connection leak issue, after a change to a pg_foreign_server
    or pg_user_mapping catalog entry, this commit makes postgres_fdw close
    the connections depending on that entry immediately if current
    transaction has not used those connections yet. Otherwise, mark those
    connections as invalid and then close them at the end of current transaction,
    since they cannot be closed in the midst of the transaction using them.
    Closed connections will be remade at the next opportunity if necessary.

    Back-patch to all supported branches.

    Author: Bharath Rupireddy
    Reviewed-by: Zhihong Yu, Zhijie Hou, Fujii Masao
    Discussion: https://postgr.es/m/CALj2ACVNcGH_6qLY-4_tXz8JLvA+4yeBThRfxMz7Oxbk1aHcpQ@mail.gmail.com

```
