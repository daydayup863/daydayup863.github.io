---
title: postgresql.git log 阅读记录
date: 2020-08-14 19:41:25
tags: 
- PostgreSQL
- git
- postgresql.git
categories: 
- PostgreSQL
top: 3
description: 
password: 

---

#  psql增加\d[i|m|t]+
```
commit 07f386ede026ae8c3f2adeba0c22139df19bf2ff
Author: Michael Paquier <michael@paquier.xyz>
Date:   Wed Sep 2 16:59:22 2020 +0900

    Add access method names to \d[i|m|t]+ in psql

    Listing a full set of relations with those psql meta-commands, without a
    matching pattern, has never showed the access method associated with
    each relation.  This commit adds the access method of tables, indexes
    and matviews, masking it for relation kinds where it does not apply.

    Note that when HIDE_TABLEAM is enabled, the information does not show
    up.  This is available when connecting to a backend version of at least
    12, where table AMs have been introduced.

    Author: Georgios Kokolatos
    Reviewed-by: Vignesh C, Michael Paquier, Justin Pryzby
    Discussion: https://postgr.es/m/svaS1VTOEscES9CLKVTeKItjJP1EEJuBhTsA0ESOdlnbXeQSgycYwVlliL5zt8Jwcfo4ATYDXtEqsExxjkSkkhCSTCL8fnRgaCAJdr0unUg=@protonmail.com

```

# 新增函数string_to_table
```
Add string_to_table() function.
    This splits a string at occurrences of a delimiter.  It is exactly like
    string_to_array() except for producing a set of values instead of an
    array of values.  Thus, the relationship of these two functions is
    the same as between regexp_split_to_table() and regexp_split_to_array().

    Although the same results could be had from unnest(string_to_array()),
    this is somewhat faster than that, and anyway it seems reasonable to
    have it for symmetry with the regexp functions.

    Pavel Stehule, reviewed by Peter Smith


```

# 逻辑复制增加正在处理的事物中的复制
```
commit 464824323e57dc4b397e8b05854d779908b55304
Author: Amit Kapila <akapila@postgresql.org>
Date:   Thu Sep 3 07:54:07 2020 +0530

    Add support for streaming to built-in logical replication.

    To add support for streaming of in-progress transactions into the
    built-in logical replication, we need to do three things:

    * Extend the logical replication protocol, so identify in-progress
    transactions, and allow adding additional bits of information (e.g.
    XID of subtransactions).

    * Modify the output plugin (pgoutput) to implement the new stream
    API callbacks, by leveraging the extended replication protocol.

    * Modify the replication apply worker, to properly handle streamed
    in-progress transaction by spilling the data to disk and then
    replaying them on commit.

    We however must explicitly disable streaming replication during
    replication slot creation, even if the plugin supports it. We
    don't need to replicate the changes accumulated during this phase,
    and moreover we don't have a replication connection open so we
    don't have where to send the data anyway.

    Author: Tomas Vondra, Dilip Kumar and Amit Kapila
    Reviewed-by: Amit Kapila, Kuntal Ghosh and Ajin Cherian
    Tested-by: Neha Sharma, Mahendra Singh Thalor and Ajin Cherian
    Discussion: https://postgr.es/m/688b0b7f-2f6c-d827-c27b-216a8e3ea700@2ndquadrant.com

```

# 新增视图pg_backend_memory_contexts
```
commit 3e98c0bafb28de87ae095b341687dc082371af54 (HEAD -> master, origin/master, origin/HEAD)
Author: Fujii Masao <fujii@postgresql.org>
Date:   Wed Aug 19 15:34:43 2020 +0900

    Add pg_backend_memory_contexts system view.

    This view displays the usages of all the memory contexts of the server
    process attached to the current session. This information is useful to
    investigate the cause of backend-local memory bloat.

    This information can be also collected by calling
    MemoryContextStats(TopMemoryContext) via a debugger. But this technique
    cannot be uesd in some environments because no debugger is available there.
    And it outputs lots of text messages and it's not easy to analyze them.
    So, pg_backend_memory_contexts view allows us to access to backend-local
    memory contexts information more easily.

    Bump catalog version.

    Author: Atsushi Torikoshi, Fujii Masao
    Reviewed-by: Tatsuhito Kasahara, Andres Freund, Daniel Gustafsson, Robert Haas, Michael Paquier
    Discussion: https://postgr.es/m/72a656e0f71d0860161e0b3f67e4d771@oss.nttdata.com

```

# log_line_prefix 增加 %P 显示并行leader
```
commit b8fdee7d0ca8bd2165d46fb1468f75571b706a01
Author: Michael Paquier <michael@paquier.xyz>
Date:   Mon Aug 3 13:38:48 2020 +0900

    Add %P to log_line_prefix for parallel group leader

    This is useful for monitoring purposes with log parsing.  Similarly to
    pg_stat_activity, the leader's PID is shown only for active parallel
    workers, minimizing the log footprint for the leaders as the equivalent
    shared memory field is set as long as a backend is alive.

    Author: Justin Pryzby
    Reviewed-by: Álvaro Herrera, Michael Paquier, Julien Rouhaud, Tom Lane
    Discussion: https://postgr.es/m/20200315111831.GA21492@telsasoft.com

```

<!-- more -->

# 增加GUC hash_mem_multiplier
```
commit d6c08e29e7bc8bc3bf49764192c4a9c71fc0b097
Author: Peter Geoghegan <pg@bowt.ie>
Date:   Wed Jul 29 14:14:58 2020 -0700

    Add hash_mem_multiplier GUC.

    Add a GUC that acts as a multiplier on work_mem.  It gets applied when
    sizing executor node hash tables that were previously size constrained
    using work_mem alone.

    The new GUC can be used to preferentially give hash-based nodes more
    memory than the generic work_mem limit.  It is intended to enable admin
    tuning of the executor's memory usage.  Overall system throughput and
    system responsiveness can be improved by giving hash-based executor
    nodes more memory (especially over sort-based alternatives, which are
    often much less sensitive to being memory constrained).

    The default value for hash_mem_multiplier is 1.0, which is also the
    minimum valid value.  This means that hash-based nodes continue to apply
    work_mem in the traditional way by default.

    hash_mem_multiplier is generally useful.  However, it is being added now
    due to concerns about hash aggregate performance stability for users
    that upgrade to Postgres 13 (which added disk-based hash aggregation in
    commit 1f39bce0).  While the old hash aggregate behavior risked
    out-of-memory errors, it is nevertheless likely that many users actually
    benefited.  Hash agg's previous indifference to work_mem during query
    execution was not just faster; it also accidentally made aggregation
    resilient to grouping estimate problems (at least in cases where this
    didn't create destabilizing memory pressure).

    hash_mem_multiplier can provide a certain kind of continuity with the
    behavior of Postgres 12 hash aggregates in cases where the planner
    incorrectly estimates that all groups (plus related allocations) will
    fit in work_mem/hash_mem.  This seems necessary because hash-based
    aggregation is usually much slower when only a small fraction of all
    groups can fit.  Even when it isn't possible to totally avoid hash
    aggregates that spill, giving hash aggregation more memory will reliably
    improve performance (the same cannot be said for external sort
    operations, which appear to be almost unaffected by memory availability
    provided it's at least possible to get a single merge pass).

    The PostgreSQL 13 release notes should advise users that increasing
    hash_mem_multiplier can help with performance regressions associated
    with hash aggregation.  That can be taken care of by a later commit.

    Author: Peter Geoghegan
    Reviewed-By: Álvaro Herrera, Jeff Davis
    Discussion: https://postgr.es/m/20200625203629.7m6yvut7eqblgmfo@alap3.anarazel.de
    Discussion: https://postgr.es/m/CAH2-WzmD%2Bi1pG6rc1%2BCjc4V6EaFJ_qSuKCCHVnH%3DoruqD-zqow%40mail.gmail.com
    Backpatch: 13-, where disk-based hash aggregation was introduced.

```

# pg_stat_statements 新增记录CREATE TABLE AS, SELECT INTO,CREATE MATERIALIZED VIEW and FETCH commands
```
commit 6023b7ea717ca04cf1bd53709d9c862db07eaefb
Author: Fujii Masao <fujii@postgresql.org>
Date:   Wed Jul 29 23:21:55 2020 +0900

    pg_stat_statements: track number of rows processed by some utility commands.

    This commit makes pg_stat_statements track the total number
    of rows retrieved or affected by CREATE TABLE AS, SELECT INTO,
    CREATE MATERIALIZED VIEW and FETCH commands.

    Suggested-by: Pascal Legrand
    Author: Fujii Masao
    Reviewed-by: Asif Rehman
    Discussion: https://postgr.es/m/1584293755198-0.post@n3.nabble.com

```

# 并行增强
```
commit 56788d2156fc32bd5737e7ac716d70e6a269b7bc
Author: David Rowley <drowley@postgresql.org>
Date:   Sun Jul 26 21:02:45 2020 +1200

    Allocate consecutive blocks during parallel seqscans

    Previously we would allocate blocks to parallel workers during a parallel
    sequential scan 1 block at a time.  Since other workers were likely to
    request a block before a worker returns for another block number to work
    on, this could lead to non-sequential I/O patterns in each worker which
    could cause the operating system's readahead to perform poorly or not at
    all.

    Here we change things so that we allocate consecutive "chunks" of blocks
    to workers and have them work on those until they're done, at which time
    we allocate another chunk for the worker.  The size of these chunks is
    based on the size of the relation.

    Initial patch here was by Thomas Munro which showed some good improvements
    just having a fixed chunk size of 64 blocks with a simple ramp-down near
    the end of the scan. The revisions of the patch to make the chunk size
    based on the relation size and the adjusted ramp-down in powers of two was
    done by me, along with quite extensive benchmarking to determine the
    optimal chunk sizes.

    For the most part, benchmarks have shown significant performance
    improvements for large parallel sequential scans on Linux, FreeBSD and
    Windows using SSDs.  It's less clear how this affects the performance of
    cloud providers.  Tests done so far are unable to obtain stable enough
    performance to provide meaningful benchmark results.  It is possible that
    this could cause some performance regressions on more obscure filesystems,
    so we may need to later provide users with some ability to get something
    closer to the old behavior.  For now, let's leave that until we see that
    it's really required.

    Author: Thomas Munro, David Rowley
    Reviewed-by: Ranier Vilela, Soumyadeep Chakraborty, Robert Haas
    Reviewed-by: Amit Kapila, Kirk Jamison
    Discussion: https://postgr.es/m/CA+hUKGJ_EErDv41YycXcbMbCBkztA34+z1ts9VQH+ACRuvpxig@mail.gmail.com

```

# GUC enable_hashagg_disk
```
commit bcbf9446a2983b6452c19cc50050456be262f7c5
Author: Peter Geoghegan <pg@bowt.ie>
Date:   Mon Jul 27 17:53:19 2020 -0700

    Remove hashagg_avoid_disk_plan GUC.

    Note: This GUC was originally named enable_hashagg_disk when it appeared
    in commit 1f39bce0, which added disk-based hash aggregation.  It was
    subsequently renamed in commit 92c58fd9.

    Author: Peter Geoghegan
    Reviewed-By: Jeff Davis, Álvaro Herrera
    Discussion: https://postgr.es/m/9d9d1e1252a52ea1bad84ea40dbebfd54e672a0f.camel%40j-davis.com
    Backpatch: 13-, where disk-based hash aggregation was introduced.

```

# logical decoding output plugin API
```
commit 45fdc9738b36d1068d3ad8fdb06436d6fd14436b
Author: Amit Kapila <akapila@postgresql.org>
Date:   Tue Jul 28 08:06:44 2020 +0530

    Extend the logical decoding output plugin API with stream methods.

    This adds seven methods to the output plugin API, adding support for
    streaming changes of large in-progress transactions.

    * stream_start
    * stream_stop
    * stream_abort
    * stream_commit
    * stream_change
    * stream_message
    * stream_truncate

    Most of this is a simple extension of the existing methods, with
    the semantic difference that the transaction (or subtransaction)
    is incomplete and may be aborted later (which is something the
    regular API does not really need to deal with).

    This also extends the 'test_decoding' plugin, implementing these
    new stream methods.

    The stream_start/start_stop are used to demarcate a chunk of changes
    streamed for a particular toplevel transaction.

    This commit simply adds these new APIs and the upcoming patch to "allow
    the streaming mode in ReorderBuffer" will use these APIs.

    Author: Tomas Vondra, Dilip Kumar, Amit Kapila
    Reviewed-by: Amit Kapila
    Tested-by: Neha Sharma and Mahendra Singh Thalor
    Discussion: https://postgr.es/m/688b0b7f-2f6c-d827-c27b-216a8e3ea700@2ndquadrant.com

```

# wal_keep_segments 改名为 wal_keep_size.
``` 
commit c3fe108c025e4a080315562d4c15ecbe3f00405e
Author: Fujii Masao <fujii@postgresql.org>
Date:   Mon Jul 20 13:30:18 2020 +0900

    Rename wal_keep_segments to wal_keep_size.

    max_slot_wal_keep_size that was added in v13 and wal_keep_segments are
    the GUC parameters to specify how much WAL files to retain for
    the standby servers. While max_slot_wal_keep_size accepts the number of
    bytes of WAL files, wal_keep_segments accepts the number of WAL files.
    This difference of setting units between those similar parameters could
    be confusing to users.

    To alleviate this situation, this commit renames wal_keep_segments to
    wal_keep_size, and make users specify the WAL size in it instead of
    the number of WAL files.

    There was also the idea to rename max_slot_wal_keep_size to
    max_slot_wal_keep_segments, in the discussion. But we have been moving
    away from measuring in segments, for example, checkpoint_segments was
    replaced by max_wal_size. So we concluded to rename wal_keep_segments
    to wal_keep_size.

    Back-patch to v13 where max_slot_wal_keep_size was added.

    Author: Fujii Masao
    Reviewed-by: Álvaro Herrera, Kyotaro Horiguchi, David Steele
    Discussion: https://postgr.es/m/574b4ea3-e0f9-b175-ead2-ebea7faea855@oss.nttdata.com
```

# 增加 generic_plans and custom_plans 域到pg_prepared_statements 
```
commit d05b172a760e0ccb3008a2144f96053720000b12
Author: Fujii Masao <fujii@postgresql.org>
Date:   Mon Jul 20 11:55:50 2020 +0900

    Add generic_plans and custom_plans fields into pg_prepared_statements.

    There was no easy way to find how many times generic and custom plans
    have been executed for a prepared statement. This commit exposes those
    numbers of times in pg_prepared_statements view.

    Author: Atsushi Torikoshi, Kyotaro Horiguchi
    Reviewed-by: Tatsuro Yamada, Masahiro Ikeda, Fujii Masao
    Discussion: https://postgr.es/m/CACZ0uYHZ4M=NZpofH6JuPHeX=__5xcDELF8hT8_2T+R55w4RQw@mail.gmail.com
```

# 逻辑复制增强， 允许二进制传输数据.
```
commit 9de77b5453130242654ff0b30a551c9c862ed661
Author: Tom Lane <tgl@sss.pgh.pa.us>
Date:   Sat Jul 18 12:44:51 2020 -0400

    Allow logical replication to transfer data in binary format.

    This patch adds a "binary" option to CREATE/ALTER SUBSCRIPTION.
    When that's set, the publisher will send data using the data type's
    typsend function if any, rather than typoutput.  This is generally
    faster, if slightly less robust.

    As committed, we won't try to transfer user-defined array or composite
    types in binary, for fear that type OIDs won't match at the subscriber.
    This might be changed later, but it seems like fit material for a
    follow-on patch.

    Dave Cramer, reviewed by Daniel Gustafsson, Petr Jelinek, and others;
    adjusted some by me

    Discussion: https://postgr.es/m/CADK3HH+R3xMn=8t3Ct+uD+qJ1KD=Hbif5NFMJ+d5DkoCzp6Vgw@mail.gmail.com
```

#  huge_page_size
```
commit d2bddc2500fb74d56e5bc53a1cfa269e2e846510
Author: Thomas Munro <tmunro@postgresql.org>
Date:   Fri Jul 17 14:33:00 2020 +1200

    Add huge_page_size setting for use on Linux.

    This allows the huge page size to be set explicitly.  The default is 0,
    meaning it will use the system default, as before.

    Author: Odin Ugedal <odin@ugedal.com>
    Discussion: https://postgr.es/m/20200608154639.20254-1-odin%40ugedal.com

```

#  jsonpath 不允许NaN
```
commit df646509f371069c65f84309eb5749642e8650b3
Author: Alexander Korotkov <akorotkov@postgresql.org>
Date:   Sat Jul 11 03:21:00 2020 +0300

    Forbid numeric NaN in jsonpath

    SQL standard doesn't define numeric Inf or NaN values.  It appears even more
    ridiculous to support then in jsonpath assuming JSON doesn't support these
    values as well.  This commit forbids returning NaN from .double(), which was
    previously allowed.  NaN can't be result of inner-jsonpath computation over
    non-NaNs.  So, we can not expect NaN in the jsonpath output.

    Reported-by: Tom Lane
    Discussion: https://postgr.es/m/203949.1591879542%40sss.pgh.pa.us
    Author: Alexander Korotkov
    Reviewed-by: Tom Lane
    Backpatch-through: 12

```
