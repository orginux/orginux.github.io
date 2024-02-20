+++
title = 'How to Compare Rows in Two CSV Files Using ClickHouse'
date = 2024-02-20T21:46:09+01:00
type = 'post'
draft = false
tags = ['clickhouse', 'snippets', 'csv']
+++

## Introduction
Dealing with data often requires comparing rows across different CSV files. In this post, we will see how to compare rows in two CSV files using ClickHouse.

Before diving into the comparison, ensure you have `clickhouse-local` installed. If not, you can download it from the [official website](https://clickhouse.tech/docs/en/getting-started/install/).

In this example, we will compare two CSV files: `stores.csv` and `stores_2.csv`. This files contain store metadata, including city, state, type, and cluster.
You can download the files from [Kaggle](https://www.kaggle.com/competitions/store-sales-time-series-forecasting/overview).
The files are the same, but `stores_2.csv` has one row less than `stores.csv`.

## Compare Two CSV Files
As always, we will use `clickhouse-local` for this kind of operation.

### Number of Rows
First, let's check the number of rows in each file:
```sql
WITH
    (
        SELECT count()
        FROM file('stores.csv')
    ) AS s1,
    (
        SELECT count()
        FROM file('stores_2.csv')
    ) AS s2
SELECT
    s1,
    s2,
    s1 = s2 AS equal
;
```

Output:
```sql
┌─s1─┬─s2─┬─equal─┐
│ 54 │ 53 │     0 │
└────┴────┴───────┘
```

As we can see, the number of rows is different.


### Find the Differences
Now, let's find the row that is in `stores.csv` but not in `stores_2.csv`:

```sql
SELECT *
FROM file('stores.csv') AS s1
LEFT JOIN file('stores_2.csv') AS s2 ON s1.store_nbr = s2.store_nbr
WHERE s2.store_nbr IS NULL
```

Output:
```sql
┌─store_nbr─┬─city──┬─state─────┬─type─┬─cluster─┬─s2.store_nbr─┬─s2.city─┬─s2.state─┬─s2.type─┬─s2.cluster─┐
│        10 │ Quito │ Pichincha │ C    │      15 │         ᴺᵁᴸᴸ │ ᴺᵁᴸᴸ    │ ᴺᵁᴸᴸ     │ ᴺᵁᴸᴸ    │       ᴺᵁᴸᴸ │
└───────────┴───────┴───────────┴──────┴─────────┴──────────────┴─────────┴──────────┴─────────┴────────────┘
```

### Store the Differences
If needed, we can store the differences in a table:
Copy
```sql
CREATE TABLE differences
ENGINE = Memory AS
SELECT *
FROM file('stores.csv') AS s1
LEFT JOIN file('stores_2.csv') AS s2 ON s1.store_nbr = s2.store_nbr
WHERE s2.store_nbr IS NULL
;
```

Or into a file:
```sql
SELECT *
FROM file('stores.csv') AS s1
LEFT JOIN file('stores_2.csv') AS s2 ON s1.store_nbr = s2.store_nbr
WHERE s2.store_nbr IS NULL
INTO OUTFILE 'differences.csv'
FORMAT CSV
```

That's it, now you know how to compare rows in two CSV files using ClickHouse.
