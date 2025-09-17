### Check Version
```
SELECT version();
```

## Delete ZooKeeper Path
```
SYSTEM DROP REPLICA 'replica-01' FROM ZKPATH '/clickhouse/tables/repl_aws_cmp_aggregation/01';
SYSTEM DROP REPLICA 'replica-02' FROM ZKPATH '/clickhouse/tables/repl_aws_cmp_aggregation/01';
SYSTEM DROP REPLICA 'replica-01' FROM ZKPATH '/clickhouse/tables/repl_aws_cmp_aggregation/02';
SYSTEM DROP REPLICA 'replica-02' FROM ZKPATH '/clickhouse/tables/repl_aws_cmp_aggregation/02';
```

## Init Database
```
DROP DATABASE tedbase1 ON CLUSTER 'ted_cluster';
DROP DATABASE tedbase10;
DROP DATABASE tedbase2;
DROP DATABASE tedbase3;
```


## CASE1 - Replicated table on cluster

```
 * Create a table be replicated each other in different database on different nodes
 * Create tables in different databases on different nodes to replicate with each other
 * 
 * Cluster: ted_cluster1 (2 Shards, 1 Replica for each shard)
 * Database: tedbase1
 * Table Engine: ReplicatedMergeTree
 */

CREATE DATABASE IF NOT EXISTS tedbase1 ON CLUSTER 'ted_cluster';
CREATE TABLE IF NOT EXISTS tedbase1.repl_aws_cmp_aggregation ON CLUSTER 'ted_cluster'
(
    `lineitem_usagestartdate` DateTime,
    `PayerAccountId` String,
    `LineItem_UsageAccountId` String,
    `Product_servicename` String,
    `Product_region` String,
    `LineItem_UsageType` String,
    `LineItem_UnblendedCost` Float32,
    `LineItem_UsageAmount` Float32,
    `Bill_BillType` String,
    `Product_ProductName` String,
    `Bill_BillingEntity` String
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/repl_aws_cmp_aggregation/{shard}', '{replica}')
PARTITION BY toYYYYMM(lineitem_usagestartdate)
ORDER BY lineitem_usagestartdate
SETTINGS index_granularity=8192;


SELECT * FROM system.zookeeper WHERE path IN ('/', '/clickhouse','/clickhouse/tables/repl_aws_cmp_aggregation') ORDER BY name;
SELECT * FROM system.zookeeper WHERE path IN ('/', '/clickhouse','/clickhouse/tables/repl_aws_cmp_aggregation/01');
SELECT * FROM `system`.zookeeper WHERE path IN ('/clickhouse/tables/repl_aws_cmp_aggregation/04');

Populate to the table

```

## CASE2 - Do execute this script to create distributed engine based on mergetree table

```
 * create distributred accrossing two tables 
 * write it to distibuted engine for distirubiting documents (>java app)
 * */
CREATE TABLE IF NOT EXISTS tedbase1.dist_repl_aws_cmp_aggregation ON CLUSTER 'ted_cluster'
(
    `lineitem_usagestartdate` DateTime,
    `PayerAccountId` String,
    `LineItem_UsageAccountId` String,
    `Product_servicename` String,
    `Product_region` String,
    `LineItem_UsageType` String,
    `LineItem_UnblendedCost` Float32,
    `LineItem_UsageAmount` Float32,
    `Bill_BillType` String,
    `Product_ProductName` String,
    `Bill_BillingEntity` String
)
ENGINE = Distributed('ted_cluster', 'tedbase1', 'repl_aws_cmp_aggregation', rand());


SELECT * from tedbase1.dist_repl_aws_cmp_aggregation;
SELECT COUNT(*) FROM tedbase1.dist_repl_aws_cmp_aggregation;
```



## CASE3 - MergeTree on the other cluster ted_cluster2

```
 * Create two tables for replication each other in different databases
 * Cluster: ted_cluster2 (1 Shards, 1 Replica of each shard)
 * Database: tedbase3
 * Table Engine: MergeTree

DROP DATABASE tedbase3;
CREATE DATABASE IF NOT EXISTS tedbase3 ON CLUSTER "ted_cluster2";
CREATE TABLE IF NOT EXISTS tedbase3.repl_aws_cmp_aggregation ON CLUSTER 'ted_cluster2'
(
    `lineitem_usagestartdate` DateTime,
    `PayerAccountId` String,
    `LineItem_UsageAccountId` String,
    `Product_servicename` String,
    `Product_region` String,
    `LineItem_UsageType` String,
    `LineItem_UnblendedCost` Float32,
    `LineItem_UsageAmount` Float32,
    `Bill_BillType` String,
    `Product_ProductName` String,
    `Bill_BillingEntity` String
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(lineitem_usagestartdate)
ORDER BY lineitem_usagestartdate
SETTINGS index_granularity=8192;


Distributed Table on mergetree across four nodes {ted-ch1..4}
CREATE TABLE IF NOT EXISTS tedbase3.dist_repl_aws_cmp_aggregation ON CLUSTER 'ted_cluster2'
(
    `lineitem_usagestartdate` DateTime,
    `PayerAccountId` String,
    `LineItem_UsageAccountId` String,
    `Product_servicename` String,
    `Product_region` String,
    `LineItem_UsageType` String,
    `LineItem_UnblendedCost` Float32,
    `LineItem_UsageAmount` Float32,
    `Bill_BillType` String,
    `Product_ProductName` String,
    `Bill_BillingEntity` String
)
ENGINE = Distributed('ted_cluster2', 'tedbase3', 'repl_aws_cmp_aggregation', rand());

SELECT COUNT(*) FROM tedbase3.test t;


//view on distributed table
CREATE VIEW IF NOT EXISTS tedbase3.view_repl_aws_cmp_aggregation ON CLUSTER 'ted_cluster2' AS SELECT * FROM tedbase3.test;
SELECT COUNT(*) FROM tedbase3.view_repl_aws_cmp_aggregation vraca ;

```		

	

	
## CASE3

```
 * Create two tables for replication each other in different databases
 * Cluster: ted_cluster2 (1 Shards, 1 Replica of each shard)
 * Database: tedbase3
 * Table Engine: MergeTree
 */
 

DROP DATABASE tedbase3;
CREATE DATABASE IF NOT EXISTS tedbase3;
CREATE TABLE IF NOT EXISTS tedbase3.repl_aws_cmp_aggregation ON CLUSTER 'ted_cluster3'
(
    `lineitem_usagestartdate` DateTime,
    `PayerAccountId` String,
    `LineItem_UsageAccountId` String,
    `Product_servicename` String,
    `Product_region` String,
    `LineItem_UsageType` String,
    `LineItem_UnblendedCost` Float32,
    `LineItem_UsageAmount` Float32,
    `Bill_BillType` String,
    `Product_ProductName` String,
    `Bill_BillingEntity` String
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(lineitem_usagestartdate)
ORDER BY lineitem_usagestartdate
SETTINGS index_granularity=8192;
```


## CASE4 Dedup
 
```
SELECT * from system.clusters;
CREATE DATABASE IF NOT EXISTS tedbase22 ON CLUSTER 'ted_cluster0';
CREATE TABLE IF NOT EXISTS tedbase22.dedup_table ON CLUSTER 'ted_cluster0'
(
	lineitem_usagestartdate DateTime, PayerAccountId String,
	LineItem_UsageAccountId String,   Product_servicename String,
	Product_region String,            LineItem_UsageType String,
	LineItem_UnblendedCost Float32,   LineItem_UsageAmount Float32,
	Bill_BillType String,             Product_ProductName String,
	Bill_BillingEntity String
)
Engine = ReplacingMergeTree
PARTITION BY toYYYYMM(lineitem_usagestartdate)
ORDER BY lineitem_usagestartdate
SETTINGS index_granularity = 8192;
```


## CASE5 go-client
```
SELECT * FROM tedbase1.repl_aws_cmp_aggregation;
SELECT * FROM tedbase1.repl_aws_cmp_aggregation limit 10;
SELECT * FROM repl_aws_cmp_aggregation
```
