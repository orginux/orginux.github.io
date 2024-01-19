+++
title = 'Convert CSV to Parquet Using ClickHouse'
date = 2024-01-16T20:56:56+01:00
tags = ['snippets', 'clickhouse']
type = 'post'
+++

## Download a Parquet file from S3

To convert a CSV to Parquet, we can use clickhouse-local:
```bash
clickhouse-local
```

And then run the following query:
```sql
SELECT *
FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/house_parquet/house_0.parquet')
INTO OUTFILE 'house_0.csv'
FORMAT CSV
```

Or just use one bash command:
```bash
clickhouse-local --query="SELECT * FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/house_parquet/house_0.parquet') INTO OUTFILE 'house_0.csv' FORMAT CSV"
```

## Local Parquet to CSV

Check the parquet file:
```bash
❯ file house_0.parquet
house_0.parquet: Apache Parquet
```

Again, we can use clickhouse-local:
```bash
clickhouse-local
```

And then run the following query:
```sql
SELECT *
FROM `house_0.parquet`
INTO OUTFILE 'house_0.csv'
FORMAT CSV
```


Check the CSV file:
```bash
❯ wc -l house_0.csv
2772030 house_0.csv
```

Or as always, we can use one bash command:
```bash
clickhouse-local --query="SELECT * FROM 'house_0.parquet' INTO OUTFILE 'house_0.csv' FORMAT CSV"
```

## Convert CSV to Parquet

Of course, we can also convert a CSV to Parquet:
```bash
clickhouse-local --query="SELECT * FROM 'house_0.csv' INTO OUTFILE 'house_1.parquet' FORMAT Parquet"
```

Check the result:
```bash
❯ file house_1.parquet
house_1.parquet: Apache Parquet
```
