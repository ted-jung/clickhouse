
# How to insert (CSV, Parquet)
## use table function from different source(S3, File, Kafka, etc)
```
> download hackernews.json.gz into current directory (Ted-lab/sample-data)

> cd Ted-lab/CH

> ./clickhouse client

> INSERT INTO hackernews FROM INFILE './user_files/hacknernews.json.gz' FORMAT JSONEachRow

> INSERT INTO hackernews2 FROM INFILE '../sample-data/hacknernews.csv' FORMAT CSV

> INSERT INTO default.hackernews3 FROM INFILE '../sample-data/hackernews.csv' FORMAT CSV;

> INSERT INTO default.uk_price_paid FROM INFILE '../sample-data/uk_property_prices.snappy.parquet' FORMAT PARQUET
```

## use insert from select 
```
> insert into rates_monthly3 
  	select toDate(parseDateTime(a,'%d/%m/%Y')),b,c,d
	from(
		select a,b,c,d
		from file('mortgage_rates.csv', 'CSV', 'a String, b Decimal32(2), c Decimal32(2), d Decimal32(2)')
	)
  SETTINGS input_format_csv_skip_first_lines=1;
```

# How to insert using cat
```
> cat covid19.csv | ./clickhouse client --query="INSERT INTO covid19db.daily_totals FORMAT CSV"
```

# How to insert (from file, under users_file)
```
> INSERT INTO operating_budget
    WITH
        splitByChar('(', c4) AS result
    SELECT 
        c1 AS fiscal_year,
        c2 AS service,
        c3 AS department,
        result[1] AS program,
        splitByChar(')',result[2])[1] AS program_code,
        c5 AS description,
        c6 AS item_category,
        toUInt32OrZero(c7) AS approved_amount,
        toUInt32OrZero(c8) AS recommended_amount,
        toDecimal64(c9, 2) AS actual_amount,
        c10 AS fund,
        c11 AS fund_type
    FROM file('operating_budget.csv', 'CSV', 'c1 String, c2 String, c3 String, c4 String, c5 String, c6 String, c7 String, c8 String, c9 String, c10 String, c11 String') 
   SETTINGS format_csv_delimiter = '~', 
            input_format_csv_skip_first_lines = 1

```

# Using clickhouse-client
```
echo 1,A > input_1.csv ; echo 2,B > input_2.csv
clickhouse-client --query="CREATE TABLE infile_globs (id UInt32, text String) ENGINE=MergeTree() ORDER BY id;"
clickhouse-client --query="INSERT INTO infile_globs FROM INFILE 'input_*.csv' FORMAT CSV;"
clickhouse-client --query="SELECT * FROM infile_globs FORMAT PrettyCompact;"
```

# Insert after selecting from S3

```
INSERT INTO trips
SELECT * FROM s3(
    'https://datasets-documentation.s3.eu-west-3.amazonaws.com/nyc-taxi/trips_{1..2}.gz',
    'TabSeparatedWithNames', "
    `trip_id` UInt32,
    `vendor_id` Enum8('1' = 1, '2' = 2, '3' = 3, '4' = 4, 'CMT' = 5, 'VTS' = 6, 'DDS' = 7, 'B02512' = 10, 'B02598' = 11, 'B02617' = 12, 'B02682' = 13, 'B02764' = 14, '' = 15),
    `pickup_date` Date,
    `pickup_datetime` DateTime,
    `dropoff_date` Date,
    `dropoff_datetime` DateTime,
    `store_and_fwd_flag` UInt8,
    `rate_code_id` UInt8,
    `pickup_longitude` Float64,
    `pickup_latitude` Float64,
    `dropoff_longitude` Float64,
    `dropoff_latitude` Float64,
    `passenger_count` UInt8,
    `trip_distance` Float64,
    `fare_amount` Float32,
    `extra` Float32,
    `mta_tax` Float32,
    `tip_amount` Float32,
    `tolls_amount` Float32,
    `ehail_fee` Float32,
    `improvement_surcharge` Float32,
    `total_amount` Float32,
    `payment_type` Enum8('UNK' = 0, 'CSH' = 1, 'CRE' = 2, 'NOC' = 3, 'DIS' = 4),
    `trip_type` UInt8,
    `pickup` FixedString(25),
    `dropoff` FixedString(25),
    `cab_type` Enum8('yellow' = 1, 'green' = 2, 'uber' = 3),
    `pickup_nyct2010_gid` Int8,
    `pickup_ctlabel` Float32,
    `pickup_borocode` Int8,
    `pickup_ct2010` String,
    `pickup_boroct2010` String,
    `pickup_cdeligibil` String,
    `pickup_ntacode` FixedString(4),
    `pickup_ntaname` String,
    `pickup_puma` UInt16,
    `dropoff_nyct2010_gid` UInt8,
    `dropoff_ctlabel` Float32,
    `dropoff_borocode` UInt8,
    `dropoff_ct2010` String,
    `dropoff_boroct2010` String,
    `dropoff_cdeligibil` String,
    `dropoff_ntacode` FixedString(4),
    `dropoff_ntaname` String,
    `dropoff_puma` UInt16
") SETTINGS input_format_try_infer_datetimes = 0
```
