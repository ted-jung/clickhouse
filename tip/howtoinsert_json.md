# How to Insert - JSON format data

## DDL with JSON type

```
  CREATE TABLE default.ted
  (
      `col1` Nullable(Int64),
      `col2` Nullable(String),
      `col3` Tuple(
          sub1 Nullable(String),
          sub2 Nullable(String)
      `col4` Int64
      `col5` JSON,
      `col6` Nullable(DateTime64(9)),
      `col7` Nullable(String),
      `col8` Nullable(String)
  )
  ENGINE = MergeTree
  ORDER BY tuple()
  SETTINGS index_granularity = 8192;
```

## sample data

col5 - Json type(not a String, Don't wrap it in double quotes)
```
  {
    "col1": 10000,
    "col2": "true",
    "col3": {
      "sub1": "",
      "sub2": "9999999999"
    },
    "col4": 1700000000000,
    "col5": {"sub3":"KRW","sub4":[{"sub4_1":"0000001","sub4_2":"null"},{"sub4_1":"0000002","sub4_2":"null"}]},
    "col6": "2025-01-01 00:00:00.000",
    "col7": "view_item",
    "col8": "POST"
  }
  {
    "col1": 10000,
    "col2": "true",
    "col3": {
      "sub1": "",
      "sub2": "9999999999"
    },
    "col4": 1700000000000,
    "col5": {"sub3":"KRW","sub4":[{"sub4_1":"0000001","sub4_2":"null"},{"sub4_1":"0000002","sub4_2":"null"}]},
    "col6": "2025-01-01 00:00:00.000",
    "col7": "view_item",
    "col8": "POST"
  }
```

## Insert

```
  INSERT INTO default.ted select * from file('ted_sample.json','JSON')
```

## Select

```
  > select col5.sub4[][] from ted
  > select col5.sub4[].sub4_1 from ted
  > select col5.sub5.:`Array(JSON)`.sub4_1 from ted
```
