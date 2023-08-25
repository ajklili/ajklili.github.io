---
layout: post
title:  "Cassandra: The Definition Guide (2)"
date:   2018-05-01
categories: technology
tags: Cassandra NoSQL
excerpt: "Reading notes for \"Cassandra: The Definition Guide\""
mathjax: true
---

* content
{:toc}


## Cassandra database

### Naming style

They chose to name the keyspace in “snake case” (my_keyspace) as  opposed to “camel case” (MyKeyspace).

Cassandra naturally handles keyspace, table, and column names as all lowercase.

### Basic commands

Run Canssandra:

```bash
bin/cqlsh
bin/cqlsh localhost 9042
```

```sql
DESCRIBE CLUSTER;
DESCRIBE KEYSPACES;
SHOW VERSION;
```


### Structure

Basic Cassandra data structures, from small to large:

1. The timestamp: each time you write data into Cassandra, a timestamp is generated for each column value that is updated
2. The column: a name/value pair
3. The row: a container for columns referenced by a primary key
4. The table: a container for rows
5. The keyspace: a container for tables
> Cassandra’s system keyspaces: meta-databases
6. The cluster: a container for keyspaces that spans one or more nodes, sometimes called the ring, because Cassandra assigns data to nodes in the cluster by arranging them in a ring

### Table, key and index

Cassandra defines a table to be a logical division that associates similar data.
It is a sparse, multidimensional array structure.

Cassandra uses a special primary key called a composite key (or compound key) to represent wide rows, also called partitions. The composite key consists of a partition key, plus an optional set of clustering columns.

After you create a table, there is no way to modify the primary key, because this controls how data is distributed within the cluster, and even more importantly, how it is stored on disk.

Cassandra also supports an additional construct called a static column, which is for storing data that is not part of the primary key but is shared by every row in a partition.

If you try to query on column in a Cassandra table that is not part of the primary key, you’ll soon realize that this is not allowed.

A secondary index is an index on a column that is not part of the primary key. (`CREATE INDEX ON`)
Cassandra partitions data across multiple nodes, each node must maintain its own copy of a secondary index based on the data stored in partitions it owns. Thus, queries involving a secondary index typically involve more nodes, making them significantly more expensive.
> The syntax of the WHERE clause involves the following rules:

> 1. All elements of the partition key must be identified
> 2. A given clustering key may only be restricted if all previous clustering keys are restricted
> 3. The exception to this rule is the ALLOW FILTERING keyword, which allows us to omit a partition key element.


SASI (SSTable attached secondary indexes) can be created on a non-collection column defined in a table. Secondary indexes are used to query a table that uses a column that is not normally queryable, such as a non primary key column. SASI implements three types of indexes, PREFIX, CONTAINS, and SPARSE. See `org.apache.cassandra.index.sasi.SASIIndex`

### Others

* TTL

    The time to live (or TTL) is a value that Cassandra stores for each column value to indicate how long to keep the value. If we want to set TTL across an entire row, we must provide a value for every non-primary key column in our INSERT or UPDATE command.

* Locale

    You can control how cqlsh displays non-ASCII characters by setting the locale via the $LANG environment variable before running the tool.

* Special types

    1. The CQL uuid type is a Type 4 UUID, universally unique identifier
    2. ISO 8601 date formats
    3. blob, A binary large object (blob) is a colloquial computing term for an arbitrary array of bytes
    4. inet, IPv4 or IPv6 Internet address
    5. timeuuid, Type 1 UUID, which is based on the MAC address of the computer, the system time, and a sequence number used to prevent duplicates
    6. set, {}, support add operator(+)
    7. list, [], support add operator(+)
    8. map
    9. user-defined types-structure
    > CREATE TYPE address(namex typex, namey typey);
    10. (the ability to access individual attributes of a nested collection is not yet supported, because the nested collection is serialized as a single object by the implementation)

* Queue

    The queue anti-pattern serves as a reminder that any design that relies on the deletion of data is potentially a poorly performing design. Cassandra use a deleted marker. See Datastax blog [Cassandra anti-patterns: Queues and queue-like datasets](https://www.datastax.com/dev/blog/cassandra-anti-patterns-queues-and-queue-like-datasets)

* For time series

    A time series–style design would store each transaction as a timestamped row and leave the work of calculating the current balance to the application.


## Data modeling

1. Use [Chebotko diagram](https://www.slideshare.net/ArtemChebotko/worlds-best-data-modeling-tool)
2. Methodology: Query sequence -(logical data model)> tables and keys -(physical model)> table column type

### Some calculations

1. Number of values in the partition
Partition size is measured by the number of cells (values) that are stored in the partition. Cassandra’s hard limit is 2 billion cells per partition.

    $$
    N_v = N_r(N_c - N_{pk} - N_s) + N_s
    $$

    The number of values (or cells) in the partition (Nv) is equal to the number of static columns (Ns) plus the product of the number of rows (Nr) and the number of of values per row. The number of values per row is defined as the number of columns (Nc) minus the number of primary key columns (Npk) and static columns (Ns).

2. Size of a table

    $$
    S_p = \sum_i sizeOf(c_k) + \sum_j sizeOf({c_{s_j}}) + N_r \times (\sum_k sizeOf(c_{r_k}) + \sum_l sizeOf(c_{c_l})) + N_v \times sizeOf(t_{avg})
    $$

    In this formula, ck refers to partition key columns, cs to static columns, cr to regular columns, and cc to clustering columns. The term tavg refers to the average number of bytes of metadata stored per cell, such as timestamps. It is typical to use an estimate of 8 bytes for this value. We recognize the number of rows Nr and number of values Nv from our previous calculations. The sizeOf() function refers to the size in bytes of the CQL data type of each referenced column.

    $$
    S_t = N_{partitions} \times N_{replicas} \times S_p
    $$

    We will need to multiply the value obtained here by the number of partitions and the number of replicas specified by the keyspace’s replication strategy in order to determine the total required total capacity for each table.

3. Split a partition

    The technique for splitting a large partition is straightforward: add an additional column to the partition key. In most cases, moving one of the existing columns into the partition key will be sufficient. Another option is to introduce an additional column to the table to act as a sharding key, but this requires additional application logic.

### Others

* User Defined Type
    Scope of a UDT is the keyspace in which it is defined

* UDF & UDA

    Setting in `cassandra.yaml`:

        enable_user_defined_functions: true

    UDF:

        CREATE FUNCTION count_if_true(input boolean)
          RETURNS NULL ON NULL INPUT
          RETURNS int
          LANGUAGE java AS 'if (input) return 1; else return 0;';

    The UDA leverages two UDFs: a state function and an optional final function.

        CREATE FUNCTION state_count_if_true(total int, input boolean)
          RETURNS NULL ON NULL INPUT
          RETURNS int
          LANGUAGE java AS 'if (input) return total+1; else return total;';

        CREATE AGGREGATE total_available (boolean)
          SFUNC state_count_if_true
          STYPE int
          INITCOND 0;

* Materialized view

    Store pre-configured views that support queries on additional columns which are not part of the original clustering key. Materialized views simplify application development: instead of the application having to keep multiple denormalized tables in sync, Cassandra takes on the responsibility of updating views in order to keep them consistent with the base table.

    Internally, materialized view updates are implemented using batching

    You can improve it [here](https://issues.apache.org/jira/browse/CASSANDRA-9928).

* Data in csv file

    COPY hotels TO/FROM 'hotels.csv' WITH HEADER=TRUE;


* IDE
    DataStax DevCenter: a free visual schema and query IDE for creating and running CQL (Cassandra Query Language) statements
    DataStax Enterprise: https://www.datastax.com/products/datastax-studio-and-development-tools