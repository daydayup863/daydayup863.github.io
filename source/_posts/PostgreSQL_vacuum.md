---
title: PostgreSQL vacuum
date: 2020-08-20 17:30:29
tags: 
- PostgreSQL
- vacuum
categories: 
- PostgreSQL
top: 21
description: 
password: 

---
vacuum processing is a maintenance process that facilitates the persistent operation of PostgreSQL. Its two main tasks are removing dead tuples and the freezing transaction ids, both of which are briefly mentioned in Section 5.10.

To remove dead tuples, vacuum processing provides two modes, i.e. Concurrent VACUUM and Full VACUUM. Concurrent VACUUM, often simply called VACUUM, removes dead tuples for each page of the table file, and other transactions can read the table while this process is running. In contrast, Full VACUUM removes dead tuples and defragments live tuples the whole file, and other transactions cannot access tables while Full VACUUM is running.

Despite the fact that vacuum processing is essential for PostgreSQL, improving its functionality has been slow compared to other functions. For example, until version 8.0, this process had to be executed manually (with the psql utility or using the cron daemon). It was automated in 2005 when the autovacuum daemon was implemented.

Since vacuum processing involves scanning whole tables, it is a costly process. In version 8.4 (2009), the Visibility Map (VM) was introduced to improve the efficiency of removing dead tuples. In version 9.6 (2016), the freeze process was improved by enhancing the VM. 

<!-- more -->

# Concurrent VACUUM
```
(1)  FOR each table
(2)       Acquire ShareUpdateExclusiveLock lock for the target table

          /* The first block */
(3)       Scan all pages to get all dead tuples, and freeze old tuples if necessary
          PostgreSQL scans a target table to build a list of dead tuples and freeze old tuples if possible. The list is stored in maintenance_work_mem in local memory
(4)       Remove the index tuples that point to the respective dead tuples if exists

          /* The second block */
(5)       FOR each page of the table
(6)            Remove the dead tuples, and Reallocate the live tuples in the page
(7)            Update FSM and VM
           END FOR

          /* The third block */
(8)       Clean up indexes
(9)       Truncate the last page if possible
(10       Update both the statistics and system catalogs of the target table
          Release ShareUpdateExclusiveLock lock
       END FOR

        /* Post-processing */
(11)  Update statistics and system catalogs
(12)  Remove both unnecessary files and pages of the clog if possible

```

(1) Get each table from the specified tables.
(2) Acquire ShareUpdateExclusiveLock lock for the table. This lock allows reading from other transactions.
(3) Scan all pages to get all dead tuples, and freeze old tuples if necessary.
(4) Remove the index tuples that point to the respective dead tuples if exists.
(5) Do the following tasks, step (6) and (7), for each page of the table.
(6) Remove the dead tuples and Reallocate the live tuples in the page.
(7) Update both the respective FSM and VM of the target table.
(8) Clean up the indexes by the index_vacuum_cleanup()@indexam.c function.
(9) Truncate the last page if the last one does not have any tuple.
(10) Update both the statistics and the system catalogs related to vacuum processing for the target table.
(11) Update both the statistics and the system catalogs related to vacuum processing.
(12) Remove both unnecessary files and pages of the clog if possible.


# Full VACUUM

```
(1)  FOR each table
(2)       Acquire AccessExclusiveLock lock for the table
(3)       Create a new table file
(4)       FOR each live tuple in the old table
(5)            Copy the live tuple to the new table file
(6)            Freeze the tuple IF necessary
            END FOR
(7)        Remove the old table file
(8)        Rebuild all indexes
(9)        Update FSM and VM
(10)      Update statistics
            Release AccessExclusiveLock lock
       END FOR
(11)  Remove unnecessary clog files and pages if possible
```

## When should I do VACUUM FULL?
There is unfortunately no best practice when you should execute ‘VACUUM FULL’. However, the extension pg_freespacemap may give you good suggestions.

The following query shows the average freespace ratio of the table you want to know.
```
testdb=# CREATE EXTENSION pg_freespacemap;
CREATE EXTENSION

testdb=# SELECT count(*) as "number of pages",
       pg_size_pretty(cast(avg(avail) as bigint)) as "Av. freespace size",
       round(100 * avg(avail)/8192 ,2) as "Av. freespace ratio"
       FROM pg_freespace('accounts');
 number of pages | Av. freespace size | Av. freespace ratio
-----------------+--------------------+---------------------
            1640 | 99 bytes           |                1.21
(1 row)
```

As the result above, You can find that there are few free spaces.

If you delete almost tuples and execute VACUUM command, you can find that almost pages are spaces ones.
```
testdb=# DELETE FROM accounts WHERE aid %10 != 0 OR aid < 100;
DELETE 90009

testdb=# VACUUM accounts;
VACUUM

testdb=# SELECT count(*) as "number of pages",
       pg_size_pretty(cast(avg(avail) as bigint)) as "Av. freespace size",
       round(100 * avg(avail)/8192 ,2) as "Av. freespace ratio"
       FROM pg_freespace('accounts');
 number of pages | Av. freespace size | Av. freespace ratio
-----------------+--------------------+---------------------
            1640 | 7124 bytes         |               86.97
(1 row)
```

The following query inspects the freespace ratio of each page of the specified table.
```
testdb=# SELECT *, round(100 * avail/8192 ,2) as "freespace ratio"
                FROM pg_freespace('accounts');
 blkno | avail | freespace ratio
-------+-------+-----------------
     0 |  7904 |           96.00
     1 |  7520 |           91.00
     2 |  7136 |           87.00
     3 |  7136 |           87.00
     4 |  7136 |           87.00
     5 |  7136 |           87.00
....
```
After executing VACUUM FULL, you can find that the table file has been compacted. 
```
testdb=# VACUUM FULL accounts;
VACUUM
testdb=# SELECT count(*) as "number of blocks",
       pg_size_pretty(cast(avg(avail) as bigint)) as "Av. freespace size",
       round(100 * avg(avail)/8192 ,2) as "Av. freespace ratio"
       FROM pg_freespace('accounts');
 number of pages | Av. freespace size | Av. freespace ratio
-----------------+--------------------+---------------------
             164 | 0 bytes            |                0.00
(1 row)
```
