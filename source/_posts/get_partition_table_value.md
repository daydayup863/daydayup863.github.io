---
title: 获取PostgreSQL hash table value 
date: 2020-09-07 19:23:12
tags: 
- PostgreSQL
- hash key
- partition table
categories: 
- PostgreSQL
top: 30 
description: 
password: 

---

简单的修改了postgresql-12.3/src/backend/partitioning/partbounds.c代码， 创建C函数获取PostgreSQL hash 分区表hash value
<!-- more -->

# 表结构

```
postgres=# \d+ userinfo
                                        Partitioned table "public.userinfo"
  Column  |              Type              | Collation | Nullable | Default | Storage  | Stats target | Description
----------+--------------------------------+-----------+----------+---------+----------+--------------+-------------
 userid   | integer                        |           |          |         | plain    |              |
 username | character varying(64)          |           |          |         | extended |              |
 ctime    | timestamp(6) without time zone |           |          |         | plain    |              |
Partition key: HASH (userid)
Indexes:
    "idx_userinfo_userid" btree (userid)
    "idx_userinfo_username" btree (username)
Partitions: userinfo_0 FOR VALUES WITH (modulus 16, remainder 0),
            userinfo_1 FOR VALUES WITH (modulus 16, remainder 1),
            userinfo_10 FOR VALUES WITH (modulus 16, remainder 10),
            userinfo_11 FOR VALUES WITH (modulus 16, remainder 11),
            userinfo_12 FOR VALUES WITH (modulus 16, remainder 12),
            userinfo_13 FOR VALUES WITH (modulus 16, remainder 13),
            userinfo_14 FOR VALUES WITH (modulus 16, remainder 14),
            userinfo_15 FOR VALUES WITH (modulus 16, remainder 15),
            userinfo_2 FOR VALUES WITH (modulus 16, remainder 2),
            userinfo_3 FOR VALUES WITH (modulus 16, remainder 3),
            userinfo_4 FOR VALUES WITH (modulus 16, remainder 4),
            userinfo_5 FOR VALUES WITH (modulus 16, remainder 5),
            userinfo_6 FOR VALUES WITH (modulus 16, remainder 6),
            userinfo_7 FOR VALUES WITH (modulus 16, remainder 7),
            userinfo_8 FOR VALUES WITH (modulus 16, remainder 8),
            userinfo_9 FOR VALUES WITH (modulus 16, remainder 9)

postgres=#
```

# 代码
vim partvalue.c
```
#include "postgres.h"
#include "fmgr.h"
#include "funcapi.h"
#include "access/relation.h"
#include "access/table.h"
#include "access/tableam.h"
#include "catalog/partition.h"
#include "catalog/pg_inherits.h"
#include "catalog/pg_type.h"
#include "commands/tablecmds.h"
#include "executor/executor.h"
#include "miscadmin.h"
#include "nodes/makefuncs.h"
#include "nodes/nodeFuncs.h"
#include "parser/parse_coerce.h"
#include "partitioning/partbounds.h"
#include "partitioning/partdesc.h"
#include "partitioning/partprune.h"
#include "utils/builtins.h"
#include "utils/datum.h"
#include "utils/fmgroids.h"
#include "utils/hashutils.h"
#include "utils/lsyscache.h"
#include "utils/partcache.h"
#include "utils/rel.h"
#include "utils/snapmgr.h"
#include "utils/ruleutils.h"
#include "utils/syscache.h"



PG_MODULE_MAGIC;

PG_FUNCTION_INFO_V1(satisfies_hash_partition_value);
/*
 * satisfies_hash_partition
 *
 * This is an SQL-callable function for use in hash partition constraints.
 * The first three arguments are the parent table OID, modulus, and remainder.
 * The remaining arguments are the value of the partitioning columns (or
 * expressions); these are hashed and the results are combined into a single
 * hash value by calling hash_combine64.
 *
 * Returns true if remainder produced when this computed single hash value is
 * divided by the given modulus is equal to given remainder, otherwise false.
 *
 * See get_qual_for_hash() for usage.
 */
Datum
satisfies_hash_partition_value(PG_FUNCTION_ARGS)
{
	typedef struct ColumnsHashData
	{
		Oid			relid;
		int			nkeys;
		Oid			variadic_type;
		int16		variadic_typlen;
		bool		variadic_typbyval;
		char		variadic_typalign;
		Oid			partcollid[PARTITION_MAX_KEYS];
		FmgrInfo	partsupfunc[FLEXIBLE_ARRAY_MEMBER];
	} ColumnsHashData;
	Oid			parentId;
	int			modulus;
	int			remainder;
	Datum		seed = UInt64GetDatum(HASH_PARTITION_SEED);
	ColumnsHashData *my_extra;
	uint64		rowHash = 0;

	/* Return null if the parent OID, modulus, or remainder is NULL. */
	if (PG_ARGISNULL(0) || PG_ARGISNULL(1) || PG_ARGISNULL(2))
		PG_RETURN_NULL();
	parentId = PG_GETARG_OID(0);
	modulus = PG_GETARG_INT32(1);
	remainder = PG_GETARG_INT32(2);

	/* Sanity check modulus and remainder. */
	if (modulus <= 0)
		ereport(ERROR,
				(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
				 errmsg("modulus for hash partition must be a positive integer")));
	if (remainder < 0)
		ereport(ERROR,
				(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
				 errmsg("remainder for hash partition must be a non-negative integer")));
	if (remainder >= modulus)
		ereport(ERROR,
				(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
				 errmsg("remainder for hash partition must be less than modulus")));

	/*
	 * Cache hash function information.
	 */
	my_extra = (ColumnsHashData *) fcinfo->flinfo->fn_extra;
	if (my_extra == NULL || my_extra->relid != parentId)
	{
		Relation	parent;
		PartitionKey key;
		int			j;

		/* Open parent relation and fetch partition keyinfo */
		parent = try_relation_open(parentId, AccessShareLock);
		if (parent == NULL)
			PG_RETURN_NULL();
		key = RelationGetPartitionKey(parent);

		/* Reject parent table that is not hash-partitioned. */
		if (parent->rd_rel->relkind != RELKIND_PARTITIONED_TABLE ||
			key->strategy != PARTITION_STRATEGY_HASH)
			ereport(ERROR,
					(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
					 errmsg("\"%s\" is not a hash partitioned table",
							get_rel_name(parentId))));

		if (!get_fn_expr_variadic(fcinfo->flinfo))
		{
			int			nargs = PG_NARGS() - 3;

			/* complain if wrong number of column values */
			if (key->partnatts != nargs)
				ereport(ERROR,
						(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
						 errmsg("number of partitioning columns (%d) does not match number of partition keys provided (%d)",
								key->partnatts, nargs)));

			/* allocate space for our cache */
			fcinfo->flinfo->fn_extra =
				MemoryContextAllocZero(fcinfo->flinfo->fn_mcxt,
									   offsetof(ColumnsHashData, partsupfunc) +
									   sizeof(FmgrInfo) * nargs);
			my_extra = (ColumnsHashData *) fcinfo->flinfo->fn_extra;
			my_extra->relid = parentId;
			my_extra->nkeys = key->partnatts;
			memcpy(my_extra->partcollid, key->partcollation,
				   key->partnatts * sizeof(Oid));

			/* check argument types and save fmgr_infos */
			for (j = 0; j < key->partnatts; ++j)
			{
				Oid			argtype = get_fn_expr_argtype(fcinfo->flinfo, j + 3);

				if (argtype != key->parttypid[j] && !IsBinaryCoercible(argtype, key->parttypid[j]))
					ereport(ERROR,
							(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
							 errmsg("column %d of the partition key has type \"%s\", but supplied value is of type \"%s\"",
									j + 1, format_type_be(key->parttypid[j]), format_type_be(argtype))));

				fmgr_info_copy(&my_extra->partsupfunc[j],
							   &key->partsupfunc[j],
							   fcinfo->flinfo->fn_mcxt);
			}
		}
		else
		{
			ArrayType  *variadic_array = PG_GETARG_ARRAYTYPE_P(3);

			/* allocate space for our cache -- just one FmgrInfo in this case */
			fcinfo->flinfo->fn_extra =
				MemoryContextAllocZero(fcinfo->flinfo->fn_mcxt,
									   offsetof(ColumnsHashData, partsupfunc) +
									   sizeof(FmgrInfo));
			my_extra = (ColumnsHashData *) fcinfo->flinfo->fn_extra;
			my_extra->relid = parentId;
			my_extra->nkeys = key->partnatts;
			my_extra->variadic_type = ARR_ELEMTYPE(variadic_array);
			get_typlenbyvalalign(my_extra->variadic_type,
								 &my_extra->variadic_typlen,
								 &my_extra->variadic_typbyval,
								 &my_extra->variadic_typalign);
			my_extra->partcollid[0] = key->partcollation[0];

			/* check argument types */
			for (j = 0; j < key->partnatts; ++j)
				if (key->parttypid[j] != my_extra->variadic_type)
					ereport(ERROR,
							(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
							 errmsg("column %d of the partition key has type \"%s\", but supplied value is of type \"%s\"",
									j + 1,
									format_type_be(key->parttypid[j]),
									format_type_be(my_extra->variadic_type))));

			fmgr_info_copy(&my_extra->partsupfunc[0],
						   &key->partsupfunc[0],
						   fcinfo->flinfo->fn_mcxt);
		}

		/* Hold lock until commit */
		relation_close(parent, NoLock);
	}

	if (!OidIsValid(my_extra->variadic_type))
	{
		int			nkeys = my_extra->nkeys;
		int			i;

		/*
		 * For a non-variadic call, neither the number of arguments nor their
		 * types can change across calls, so avoid the expense of rechecking
		 * here.
		 */

		for (i = 0; i < nkeys; i++)
		{
			Datum		hash;

			/* keys start from fourth argument of function. */
			int			argno = i + 3;

			if (PG_ARGISNULL(argno))
				continue;

			hash = FunctionCall2Coll(&my_extra->partsupfunc[i],
									 my_extra->partcollid[i],
									 PG_GETARG_DATUM(argno),
									 seed);

			/* Form a single 64-bit hash value */
			rowHash = hash_combine64(rowHash, DatumGetUInt64(hash));
		}
	}
	else
	{
		ArrayType  *variadic_array = PG_GETARG_ARRAYTYPE_P(3);
		int			i;
		int			nelems;
		Datum	   *datum;
		bool	   *isnull;

		deconstruct_array(variadic_array,
						  my_extra->variadic_type,
						  my_extra->variadic_typlen,
						  my_extra->variadic_typbyval,
						  my_extra->variadic_typalign,
						  &datum, &isnull, &nelems);

		/* complain if wrong number of column values */
		if (nelems != my_extra->nkeys)
			ereport(ERROR,
					(errcode(ERRCODE_INVALID_PARAMETER_VALUE),
					 errmsg("number of partitioning columns (%d) does not match number of partition keys provided (%d)",
							my_extra->nkeys, nelems)));

		for (i = 0; i < nelems; i++)
		{
			Datum		hash;

			if (isnull[i])
				continue;

			hash = FunctionCall2Coll(&my_extra->partsupfunc[0],
									 my_extra->partcollid[0],
									 datum[i],
									 seed);

			/* Form a single 64-bit hash value */
			rowHash = hash_combine64(rowHash, DatumGetUInt64(hash));
		}
	}

	PG_RETURN_UINT64(rowHash % modulus);
}

```

# 编译

```
gcc -fPIC -c partvalue.c -I /home/postgres/pg12/include/server ; cc -shared -o partvalue.so partvalue.o
```

# 创建函数
```
postgres=# CREATE or replace FUNCTION satisfies_hash_partition_value(oid, integer, integer, VARIADIC "any") RETURNS bigint
     AS '/home/postgres/partvalue', 'satisfies_hash_partition_value'
     LANGUAGE C STRICT;
CREATE FUNCTION
postgres=# \df+ satisfies_hash_partition_value
                                                                                                           List of functions
 Schema |              Name              | Result data type |          Argument data types          | Type | Volatility | Parallel |  Owner   | Security | Access privileges | Language |          Source code           | Description
--------+--------------------------------+------------------+---------------------------------------+------+------------+----------+----------+----------+-------------------+----------+--------------------------------+-------------
 public | satisfies_hash_partition_value | bigint           | oid, integer, integer, VARIADIC "any" | func | volatile   | unsafe   | postgres | invoker  |                   | c        | satisfies_hash_partition_value |
(1 row)

postgres=#

```

# 测试
```
postgres=# insert into userinfo select id, 'userinfo_'|| satisfies_hash_partition_value('18544'::oid, 16, 0, id),  now() - (id || ' sec')::interval from generate_series(1,50) id;
INSERT 0 50
postgres=# select * from userinfo;
 userid |  username   |           ctime
--------+-------------+----------------------------
     14 | userinfo_0  | 2020-09-07 19:29:55.519518
     26 | userinfo_0  | 2020-09-07 19:29:43.519518
     34 | userinfo_0  | 2020-09-07 19:29:35.519518
     11 | userinfo_1  | 2020-09-07 19:29:58.519518
     19 | userinfo_1  | 2020-09-07 19:29:50.519518
     21 | userinfo_1  | 2020-09-07 19:29:48.519518
     36 | userinfo_1  | 2020-09-07 19:29:33.519518
     42 | userinfo_2  | 2020-09-07 19:29:27.519518
      4 | userinfo_3  | 2020-09-07 19:30:05.519518
      6 | userinfo_3  | 2020-09-07 19:30:03.519518
     24 | userinfo_3  | 2020-09-07 19:29:45.519518
     29 | userinfo_3  | 2020-09-07 19:29:40.519518
     44 | userinfo_4  | 2020-09-07 19:29:25.519518
     50 | userinfo_4  | 2020-09-07 19:29:19.519518
      8 | userinfo_5  | 2020-09-07 19:30:01.519518
     13 | userinfo_6  | 2020-09-07 19:29:56.519518
     23 | userinfo_6  | 2020-09-07 19:29:46.519518
     39 | userinfo_6  | 2020-09-07 19:29:30.519518
     48 | userinfo_6  | 2020-09-07 19:29:21.519518
     49 | userinfo_6  | 2020-09-07 19:29:20.519518
      7 | userinfo_7  | 2020-09-07 19:30:02.519518
     10 | userinfo_7  | 2020-09-07 19:29:59.519518
     22 | userinfo_7  | 2020-09-07 19:29:47.519518
      1 | userinfo_8  | 2020-09-07 19:30:08.519518
     16 | userinfo_8  | 2020-09-07 19:29:53.519518
     28 | userinfo_8  | 2020-09-07 19:29:41.519518
     30 | userinfo_8  | 2020-09-07 19:29:39.519518
     32 | userinfo_8  | 2020-09-07 19:29:37.519518
      3 | userinfo_9  | 2020-09-07 19:30:06.519518
     31 | userinfo_9  | 2020-09-07 19:29:38.519518
     35 | userinfo_9  | 2020-09-07 19:29:34.519518
     37 | userinfo_9  | 2020-09-07 19:29:32.519518
     38 | userinfo_9  | 2020-09-07 19:29:31.519518
      2 | userinfo_10 | 2020-09-07 19:30:07.519518
     47 | userinfo_10 | 2020-09-07 19:29:22.519518
     15 | userinfo_11 | 2020-09-07 19:29:54.519518
     12 | userinfo_12 | 2020-09-07 19:29:57.519518
     17 | userinfo_12 | 2020-09-07 19:29:52.519518
     45 | userinfo_12 | 2020-09-07 19:29:24.519518
      5 | userinfo_13 | 2020-09-07 19:30:04.519518
      9 | userinfo_13 | 2020-09-07 19:30:00.519518
     20 | userinfo_13 | 2020-09-07 19:29:49.519518
     41 | userinfo_13 | 2020-09-07 19:29:28.519518
     46 | userinfo_13 | 2020-09-07 19:29:23.519518
     18 | userinfo_14 | 2020-09-07 19:29:51.519518
     25 | userinfo_14 | 2020-09-07 19:29:44.519518
     27 | userinfo_14 | 2020-09-07 19:29:42.519518
     43 | userinfo_14 | 2020-09-07 19:29:26.519518
     33 | userinfo_15 | 2020-09-07 19:29:36.519518
     40 | userinfo_15 | 2020-09-07 19:29:29.519518
(50 rows)

postgres=# select * from userinfo_0;
 userid |  username  |           ctime
--------+------------+----------------------------
     14 | userinfo_0 | 2020-09-07 19:29:55.519518
     26 | userinfo_0 | 2020-09-07 19:29:43.519518
     34 | userinfo_0 | 2020-09-07 19:29:35.519518
(3 rows)

postgres=# select * from userinfo_1;
 userid |  username  |           ctime
--------+------------+----------------------------
     11 | userinfo_1 | 2020-09-07 19:29:58.519518
     19 | userinfo_1 | 2020-09-07 19:29:50.519518
     21 | userinfo_1 | 2020-09-07 19:29:48.519518
     36 | userinfo_1 | 2020-09-07 19:29:33.519518
(4 rows)

postgres=# select * from userinfo_2;
 userid |  username  |           ctime
--------+------------+----------------------------
     42 | userinfo_2 | 2020-09-07 19:29:27.519518
(1 row)

postgres=# select * from userinfo_3;
 userid |  username  |           ctime
--------+------------+----------------------------
      4 | userinfo_3 | 2020-09-07 19:30:05.519518
      6 | userinfo_3 | 2020-09-07 19:30:03.519518
     24 | userinfo_3 | 2020-09-07 19:29:45.519518
     29 | userinfo_3 | 2020-09-07 19:29:40.519518
(4 rows)

```
