## Execution Plan statement.


### EXPLAIN options, to be used

```
EXPLAIN PIPELINE
EXPLAIN PIPELINE graph = 1
EXPLAIN AST graph = 1
EXPLAIN PIPELINE graph = 1, compact = 0

```

---

Example1) How to use

```
EXPLAIN AST graph = 1
SELECT URL, Country(*) AS PageValues
FROM datasets.hits_v1
WHERE CounterID = 62
  AND EventDate >= '2013-07-01'
  AND EventDate <= '2013-07-31'
  AND DontCountHits =0
  AND URL <> ''
GROUP BY URL
ORDER BY PageViews DESC LIMIT 10;
```

Example2) Lets see the pipeline

```
EXPLAIN PIPELINE graph=1, compact=0
SELECT URL, COUNT(*) AS PageValues
FROM datasets.hits_v1
WHERE CounterID = 57
  AND EventDate >= '2014-01-01'
  AND EventDate <= '2014-12-31'
  AND DontCountHits =0
  AND URL <> ''
GROUP BY URL
ORDER BY PageValues DESC LIMIT 10
SETTINGS max_threads = 2
```

---

### what impacts the number of threads

In case of reading of the rows(750K rows) returned after a single query
=> pageviews: 750,000 rows

How many physical cores? (includes hyperthreads are enabled)
=> lets suppose(24 cores on the machine)

```
max_threads="number of default cores of the machine"
```

싱글쿼리 쓰레드가 최소 읽어야하는 로우갯수 
minimum number of rows that a single query execution thread should read at least

```
merge_tree_min_rows_for_concurrent_read_for_remote_filesystem
```

싱글쿼리 쓰레드가 최소 읽어야하는 최소 데이터양
minimum amount of data that a single query execution thread should read at least
```
merge_tree_min_bytes_for_concurrent_read_for_remote_filesystem
```

Total number of threads (실행시 선택되는(=계산되는 threads수))

```
total rows/merge_tree_min_rows_for_concurrent_read_for_remote_filesystem

-> Total rows: 750,000
-> merge_tree~~~: 163,840

750,000/163840 = 4

4 threads in this example having 750,000 rows
each of the thread handles 180K rows
```

