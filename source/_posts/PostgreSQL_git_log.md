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
