---
layout: post
title: "OR is slow in BigQuery JOIN statements"
author: Peter Boone
tags: ["tech", "bigquery"]
date: 2024-02-21
draft: true
---

First, create a table with 1,000,000 rows:

```sql
CREATE TABLE IF NOT EXISTS `project.dataset.table_1` AS (
  SELECT
    x
  FROM UNNEST( GENERATE_ARRAY(1, 1000000)) x
);
```

Joining this table to itself is quite fast:

```sql
SELECT
  COUNT(1)
FROM `project.dataset.table_1` a
JOIN `project.dataset.table_1` b
ON a.x = b.x
```

   |   
---|---
Duration|1 sec
Bytes processed|7.63 MB
Bytes billed|10 MB
Slot milliseconds|2326

However, adding an `OR` condition to the `JOIN` statement causes the query to fail:

```sql
SELECT
  COUNT(1)
FROM `project.dataset.table_1` a
JOIN `project.dataset.table_1` b
ON a.x = b.x
OR a.x + 1 = b.x + 1
```

> Query exceeded resource limits. This query used 122966 CPU seconds but would charge only 10M Analysis bytes. This exceeds the ratio supported by the on-demand pricing model. Please consider moving this workload to the flat-rate reservation pricing model, which does not have this limit. 122966 CPU seconds were used, and this query must use less than 2500 CPU seconds.


This query reads 80,000,000 records compared to the 2,000,000 of the first query.

```
Step details
READ $1
FROM __stage01_output
READ $10
FROM __stage00_output
AGGREGATE $30 := COUNT($40)
COMPUTE $40 := 1
FILTER or(equal($50, $51), equal(add($50, 1), add($51, 1)))
JOIN $50 := $1, $51 := $10
CROSS EACH  WITH EACH 
WRITE $30
TO __stage02_output

```