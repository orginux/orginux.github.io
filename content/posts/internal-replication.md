+++
title = 'Internal Replication setting in ClickHouse'
date = 2024-01-19T20:02:41+01:00
type = 'post'
draft = true
tags = ['clickhouse', 'settings']
+++

There are two option for `internal_replication` setting in ClickHouse:

### internal_replication = true
This option is useful when you're using Replicated tables and insert data into Distributed table (most of cases).
The Distributed table inserts data only into one of underlying table replicas, after that the Replicated table replicates data into other replicas.

### internal_replication = false (default)
This option can be useful when you're using non-Replicated tables and insert data into Distributed table.
Writes down data to all underlying replicas if we do insert in a Distributed table. In this case, the Distributed table replicates data itself.

![view](/internal_replication/global.png)

## Simple example
### Config
We have two ClickHouse clusters: CH-A and CH-B:
For shards of CH-A we set `internal_replication = false`:
```xml
            ...
            <shard>
                <internal_replication>false</internal_replication>
                <replica>
                ...
```

And for shards of CH-B we set `internal_replication = true`:
```xml
            ...
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                ...
```


You can use my [clickhouse-cluster-compose](https://github.com/orginux/clickhouse-cluster-compose) project to deploy this example.

Table sutrucure will be the same for both clusters, difference only in `internal_replication` setting.
### Create tables:
```sql
CREATE TABLE table_local ON CLUSTER '{cluster}'
(
    `num` Int32
)
ENGINE = MergeTree
ORDER BY num
;

CREATE OR REPLACE TABLE table_distr ON CLUSTER '{cluster}' AS table_local
ENGINE = Distributed('{cluster}', default, table_local, rand())
;
```

### Insert data:
On one of replicas of both clusters execute:
```sql
INSERT INTO table_distr VALUES (1)(2)(3)(4);
```

### Count inser queries:
Now we can check how many insert queries were executed on each replica:
```sql
SELECT count()
FROM remote('clickhouse-01-01', 'system.query_log')
WHERE query_kind = 'Insert';

SELECT count()
FROM remote('clickhouse-01-02', 'system.query_log')
WHERE query_kind = 'Insert';
```

On CH-A (`internal_replication = false`) we have:
```
┌─count()─┐
│       2 │
└─────────┘

┌─count()─┐
│       2 │
└─────────┘
```

On CH-B (`internal_replication = true`) the result is:
```
┌─count()─┐
│       2 │
└─────────┘

┌─count()─┐
│       0 │
└─────────┘
```

### Links
- [ClickHouse internal_replication setting](https://simpl1g.medium.com/clickhouse-internal-replication-setting-b6d8c7c2a9f2)
- [Distributed writing data](https://clickhouse.com/docs/en/engines/table-engines/special/distributed#distributed-writing-data)
- [Replication and Sharding configuration](https://clickhouse.com/docs/en/architecture/replication#replication-and-sharding-configuration)
