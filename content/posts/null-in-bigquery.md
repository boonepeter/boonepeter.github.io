---
layout: post
title: "NULL in BigQuery"
author: Peter Boone
tags: ["tech", "bigquery", "google"]
date: 2021-09-14
draft: false
---

Keeping track of how __NULL__ values are handled in different SQL dialects can be tricky.
This post will serve as a living document where I keep track of how BigQuery does this.

## `NULL` and `STRING` comparison

Comparison always returns `NULL`.

```sql
SELECT
  CAST(NULL AS STRING) = '', -- null
  CAST(NULL AS STRING) = 'value', -- null
  CAST(NULL AS STRING) = CAST(NULL AS STRING) -- null
```

## `NULL` and `BOOL` comparison

`NULL` is not `TRUE` or `FALSE`. It is `NULL`. Comparing `NULL` with `=` or `<>` is `NULL`.

```sql
SELECT 
    CAST(NULL AS BOOL) is TRUE, -- false
    CAST(NULL AS BOOL) is FALSE, -- false
    CAST(NULL AS BOOL) is NULL, -- true
    CAST(NULL AS BOOL) = TRUE, -- null
    CAST(NULL AS BOOL) = FALSE, -- null
    CAST(NULL AS BOOL) = CAST(NULL AS BOOL), -- null
    CAST(NULL AS BOOL) <> TRUE, -- null
    CAST(NULL AS BOOL) <> FALSE, -- null
    CAST(NULL AS BOOL) <> CAST(NULL AS BOOL), -- null
```

## `NULL` and `BOOL` [operators](https://cloud.google.com/bigquery/docs/reference/standard-sql/operators#logical_operators)

`NULL` and `FALSE` is False, and `NULL` or `TRUE` is True.

```sql
SELECT
  CAST(NULL AS BOOL) AND TRUE, -- null
  CAST(NULL AS BOOL) AND FALSE, -- false
  CAST(NULL AS BOOL) OR TRUE, -- true
  CAST(NULL AS BOOL) OR FALSE, -- null
  NOT CAST(NULL AS BOOL) -- null
```

## `NULL` and `NUMERIC` operations

[According to the docs](https://cloud.google.com/bigquery/docs/reference/standard-sql/mathematical_functions#:~:text=They%20return%20NULL,arguments%20is%20NaN.):
> All mathematical functions have the following behaviors:
> - They return NULL if any of the input parameters is NULL.
> - They return NaN if any of the arguments is NaN.

Which I can confirm:
```sql
SELECT 
  1 + CAST(NULL AS INT64), -- null
  1 - CAST(NULL AS INT64), -- null
  1 / CAST(NULL AS INT64), -- null
  1 * CAST(NULL AS INT64), -- null
  POW(1, CAST(NULL AS INT64)), -- null
  ABS(CAST(NULL AS INT64)), -- null
  1.1 + CAST(NULL AS FLOAT64), -- null
  1.1 - CAST(NULL AS FLOAT64), -- null
  1.1 / CAST(NULL AS FLOAT64), -- null
  1.1 * CAST(NULL AS FLOAT64), -- null
  POW(1.1, CAST(NULL AS FLOAT64)), -- null
  ABS(CAST(NULL AS FLOAT64)), -- null
```

## `NULL` and `NUMERIC` comparison

Directly comparing returns `NULL`:
```sql
WITH a AS (SELECT NULL AS n)
SELECT
  1 < n, -- null
  1 > n, -- null
  1 = n, -- null
  1 <> n, -- null
  1.1 < n, -- null
  1.1 < n, -- null
FROM
  a
```

When ordering, `NULL` is the [lowest value](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#:~:text=Floating%20point%20values%20are%20sorted%20in%20this%20order%2C%20from%20least%20to%20greatest%3A) (below `NaN` and `-infinity`):

```sql
    SELECT
        n
    FROM UNNEST([
        1, 
        -1, 
        0, 
        CAST(NULL AS INT64), 
        CAST('NaN' AS FLOAT64), 
        CAST('inf' AS FLOAT64), 
        CAST('-inf' AS FLOAT64)]) AS n
    ORDER BY n
```
returns:

Row|n
---|---
1|`null`
2|`NaN`
3|-Infinity
4|-1.0
5|0.0
6|1.0
7|Infinity



