## =============================================================================
## Title: Materialzed View
## Writter: Ted,Jung
## Date: 13, Feb 2026
## Created: 13, Feb 2026
## Updated: 13, Feb 2026
## Description: The use case of each MV features
##       Engine: MergeTree or *ReplicatedMergeTree
## =============================================================================



## Deduplication [ Insert, Insert ~ Select ]

## Setings

```
    > SELECT * FROM system.settings LIMIT 10;
    > SELECT * FROM system.server_settings WHERE name ILIKE '%replicated_%';
    > SELECT * FROM system.merge_tree_settings WHERE name ILIKE 'replicated_%';
    > SELECT * FROM system.merge_tree_settings WHERE name ILIKE 'non_replicated_%';
    > SELECT * FROM system.merge_tree_settings WHERE name ILIKE 'deduplicate_%';
```

```
    > DROP TABLE ted_dest,ted_mv,mv_first,mv_second;

    > CREATE or REPLACE TABLE ted_dest
      (
          `key` Int64,
          `value` String
      )
      ENGINE = MergeTree
      ORDER BY tuple()
      SETTINGS replicated_deduplication_window=1000;
  
    > CREATE or REPLACE TABLE ted_mv
      (
          `key` Int64,
          `value` String
      )
      ENGINE = MergeTree
      ORDER BY tuple()
      SETTINGS replicated_deduplication_window=1000;
  
  
    > CREATE MATERIALIZED VIEW mv_first
      TO ted_mv
      AS SELECT 0 AS key, value AS value
          FROM ted_dest;
  
    > CREATE MATERIALIZED VIEW mv_second
      TO ted_mv
      AS SELECT 0 AS key, value AS value
          FROM ted_dest;
```

## select 'first attempt';

```
    > INSERT INTO ted_dest SETTINGS deduplicate_blocks_in_dependent_materialized_views=1 VALUES (1, 'A');

    > SELECT 'from ted_dest', *, _part
        FROM ted_dest
    ORDER by all;

    > SELECT 'from ted_mv', *, _part
        FROM ted_mv
    ORDER by all;
```

## SELECT 'second attempt';

```
    > INSERT INTO ted_dest SETTINGS deduplicate_blocks_in_dependent_materialized_views=1 VALUES (1, 'A');

    > SELECT 'from ted_dest', *, _part
        FROM ted_dest
    ORDER by all;

    > SELECT 'from ted_mv', *, _part
        FROM ted_mv
    ORDER by all;
```
