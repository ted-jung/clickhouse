
### Memory-Intensive Operations (In-Memory)

Some key steps are designed to run in memory for maximum speed, but they often have spillover mechanisms:

Intermediate Aggregations: Small to moderate GROUP BY and aggregation results are stored in hash tables in RAM.   

Primary Index/Marks: The primary index marks for the data parts being scanned are loaded into memory for quick data skipping.   

Dictionaries: Dimensional data loaded into ClickHouse dictionaries resides entirely in RAM for fast lookups.


### Disk-Assisted Operations (Spill-to-Disk)

There are use cases to handle data that exceeds available RAM, preventing out-of-memory (OOM) errors

1. Large Aggregations:

It writes intermediate aggregation results to a temporary file on the disk and loads chunks back into memory as needed.
This is controlled by settings like

```
    - max_memory_usage
    - max_rows_to_group_by
```

2. External Sorting:

If an ORDER BY clause requires sorting a result set that is too large to fit in memory, ClickHouse performs external sorting, writing intermediate sorted chunks to disk and then merging them. This is controlled by the

```
    - max_rows_to_sort
    - max_bytes_to_sort
```

3. Complex or large JOIN operations (especially with the default HASH algorithm) can spill the right-side table to disk if it exceeds the allocated memory limit.
