+++
title = 'Selecting Data from Google Cloud Storage Using ClickHouse'
date = 2025-02-16T17:46:58+01:00
draft =  false
tags = ['clickhouse', 'gcs', 'gcp', 'snippets']
toc = false
+++


## Create keys to access the bucket from ClickHouse
```bash
gcloud storage hmac create {service-account}
```


## Use the keys to create a named collection in ClickHouse
```sql
CREATE NAMED COLLECTION bucket AS
    url = 'https://storage.googleapis.com/{bucket}/*.parquet',
    access_key_id = '{access-key-id}',
    secret_access_key = '{secret-access-key}'
;
```


## Query the data from the bucket
```sql
SELECT *
FROM gcs(bucket)
LIMIT 10
```
