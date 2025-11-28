### Create table and distributed Table with Macro

Need to define macro first on each clickhouse hosts like below

```
- host1
<macros>
    <shard>01</shard>
    <replica>01</replica>
</macros>

- host3
<macros>
    <shard>01</shard>
    <replica>02</replica>
</macros>

- host2
<macros>
    <shard>02</shard>
    <replica>01</replica>
</macros>

- host4
<macros>
    <shard>02</shard>
    <replica>02</replica>
</macros>

> SELECT * FROM system.macros;

```

Create Database and see
```
CREATE DATABASE my_db ON CLUSTER cluster1;

SHOW databases;
```

Create a table on top of cluster and create a distributed table

```
CREATE TABLE my_db.my_table ON CLUSTER cluster1
(
    ...
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/my_table/{shard}', '{replica}')
ORDER BY (..)

CREATE TABLE my_distributed_table AS my_db.my_table
ENGINE=Distributed(cluster1, my_db, my_table, rand())
```


Download a file for testing and load it.

```
> clickhouse-client --time --query "INSERT INTO my-db.hits_distributed FORMAT TSV" < hits.tsv

> SELECT formatReadableQuantity(count()) FROM my_db.hits_distributed;
> SELECT formatReadableQuantity(count()) FROM mydb.hits;

> SELECT URL, COUNT(*) AS PageViews
    FROM my_db.hits_distributed
   WHERE (CounterID = 62)
     AND (EventDate >='2013-07-01')
     AND (EventDate <='2013-07-31')
     AND (DontCountHits = 0)
     AND (IsRefresh = 0)
     ADN (URL != '')
   GROUP BY URL
   ORDER BY PageViews DESC
   LIMIT 10
```
