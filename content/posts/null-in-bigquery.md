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

