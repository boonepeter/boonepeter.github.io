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


## `NULL` and `STRING` operations

String functions return `NULL` if any of the input parameters is `NULL`.

```sql
SELECT 
    ASCII(s) as _ascii, -- null
    BYTE_LENGTH(s) as _byte_length, -- null
    CHAR_LENGTH(s) as _char_length, -- null
    CHR(NULL) as _chr, -- null
    CONCAT(s, s) as _concat, -- null
    CONCAT(s, 'test') as _concat, -- null
    CONTAINS_SUBSTR(s, 'test') as _contains_substr, -- null
    ENDS_WITH(s, s) as _ends_with, -- null
    ENDS_WITH(s, 'test') as _ends_with, -- null
    ENDS_WITH('test', s) as _ends_with, -- null
    INITCAP(s) as _initcap, -- null
    INSTR(s, s) as _instr, -- null
    INSTR('test', s) as _instr, -- null
    LEFT(s, 0) as _left, -- null
    LENGTH(s) as _length, -- null
    LPAD(s, 5, 'A') as _lpad, -- null
    LOWER(s) as _lower, -- null
    LTRIM(s) as _ltrim, -- null
    NORMALIZE(s) as _normalize, -- null
    NORMALIZE_AND_CASEFOLD(s) as _normalize_and_casefold, -- null
    OCTET_LENGTH(s) as _octet_length, -- null
    REGEXP_CONTAINS(s, 'test') as _regexp_contains, -- null
    REGEXP_CONTAINS('test', s) as _regexp_contains, -- null
    REGEXP_EXTRACT(s, 'test') as _regexp_extract, -- null
    REGEXP_EXTRACT('test', s) as _regexp_extract, -- null
    REGEXP_EXTRACT_ALL('test', s) as _regexp_extract_all, -- null
    REGEXP_EXTRACT_ALL(s, 'test') as _regexp_extract_all, -- null
    REGEXP_INSTR('test', s) as _regexp_instr, -- null
    REGEXP_INSTR(s, 'test') as _regexp_instr, -- null
    REGEXP_REPLACE(s, 'test', 'test2') as _regexp_replace, -- null
    REGEXP_REPLACE('test', s, 'test2') as _regexp_replace, -- null
    REGEXP_REPLACE('test2', 'test', s) as _regexp_replace, -- null
    REGEXP_SUBSTR(s, 'test') as _regexp_substr, -- null
    REGEXP_SUBSTR('test', s) as _regexp_substr, -- null
    REPLACE(s, 'test', 'test2') as _replace, -- null
    REPLACE('test', s, 'test2') as _replace, -- null
    REPLACE('test2', 'test', s) as _replace, -- null
    REPEAT(s, 5) as _repeat, -- null
    REVERSE(s) as _reverse, -- null
    RIGHT(s, 5) as _right, -- null
    RIGHT(s, 0) as _right, -- null
    RPAD(s, 5, 'test') as _rpad, -- null
    RTRIM(s, 'test') as _rtrim, -- null
    SOUNDEX(s) as _soundex, -- null
    SPLIT(s, ',') as _split, -- null
    STARTS_WITH(s, 'test') as _starts_with, -- null
    STRPOS(s, 'test') as _strpos, -- null
    SUBSTR(s, 0) as _substr, -- null
    TO_CODE_POINTS(s) as _to_code_points, -- null
    TRANSLATE(s, 'a', 'A') as _translate, -- null
    TRIM(s, 'test') as _trim, -- null
    TRIM('test', s) as _trim, -- null
    UNICODE(s) as _unicode, -- null
    UPPER(s) as _upper, -- null
FROM (
    SELECT CAST(NULL AS STRING) AS s
) value
```

## `NULL` and `BYTES` operations

`BYTES` operations return `NULL` if any of the input parameters is `NULL`, the same as `STRING` operations.

```sql
SELECT 
    ASCII(s) as _ascii, -- null
    BYTE_LENGTH(s) as _byte_length, -- null
    CHR(NULL) as _chr, -- null
    CONCAT(s, s) as _concat, --
    CONCAT(s, b'test') as _concat,
    CONTAINS_SUBSTR(s, 'test') as _contains_substr,
    ENDS_WITH(s, s) as _ends_with,
    ENDS_WITH(s, b'test') as _ends_with,
    ENDS_WITH(b'test', s) as _ends_with,
    INSTR(s, s) as _instr,
    INSTR(b'test', s) as _instr,
    LEFT(s, 0) as _left,
    LENGTH(s) as _length,
    LPAD(s, 5, b'A') as _lpad,
    LOWER(s) as _lower,
    OCTET_LENGTH(s) as _octet_length,
    REGEXP_CONTAINS(s, b'test') as _regexp_contains,
    REGEXP_CONTAINS(b'test', s) as _regexp_contains,
    REGEXP_EXTRACT(s, b'test') as _regexp_extract,
    REGEXP_EXTRACT(b'test', s) as _regexp_extract,
    REGEXP_EXTRACT_ALL(b'test', s) as _regexp_extract_all,
    REGEXP_EXTRACT_ALL(s, b'test') as _regexp_extract_all,
    REGEXP_INSTR(b'test', s) as _regexp_instr,
    REGEXP_INSTR(s, b'test') as _regexp_instr,
    REGEXP_REPLACE(s, b'test', b'test2') as _regexp_replace,
    REGEXP_REPLACE(b'test', s, b'test2') as _regexp_replace,
    REGEXP_REPLACE(b'test2', b'test', s) as _regexp_replace,
    REGEXP_SUBSTR(s, b'test') as _regexp_substr,
    REGEXP_SUBSTR(b'test', s) as _regexp_substr,
    REPLACE(s, b'test', b'test2') as _replace,
    REPLACE(b'test', s, b'test2') as _replace,
    REPLACE(b'test2', b'test', s) as _replace,
    REPEAT(s, 5) as _repeat,
    REVERSE(s) as _reverse,
    RIGHT(s, 5) as _right,
    RIGHT(s, 0) as _right,
    RPAD(s, 5, b'test') as _rpad,
    RTRIM(s, b'test') as _rtrim,
    SPLIT(s, b',') as _split,
    STARTS_WITH(s, b'test') as _starts_with,
    STRPOS(s, b'test') as _strpos,
    SUBSTR(s, 0) as _substr,
    TO_CODE_POINTS(s) as _to_code_points,
    TRANSLATE(s, b'a', b'A') as _translate,
    TRIM(s, b'test') as _trim,
    TRIM(b'test', s) as _trim,
    UPPER(s) as _upper,
FROM (
    SELECT CAST(NULL AS BYTES) AS s
) value
```


## `NULL` and `ARRAY` types

BigQuery has [two limitations](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#:~:text=Currently%2C%20BigQuery%20has,two%20distinct%20values.)
 for `NULL`s and `ARRAY`s:

> BigQuery raises an error if query result has ARRAYs which contain NULL elements, although such ARRAYs can be used inside the query.

> BigQuery translates NULL ARRAY into empty ARRAY in the query result, although inside the query NULL and empty ARRAYs are two distinct values.

So while this query works:

```sql
SELECT
    -- value,
    ARRAY_LENGTH(value) as _array_length,
FROM (
    SELECT
        [NULL] as value
)
```

If you uncomment the `value` line, you will get an error:

> Array cannot have a null element; error in writing field value

Here is a comparison between `NULL` and empty arrays:

```sql
WITH a AS (
  SELECT
    [] AS value,
    'empty_array' as value_type
  UNION ALL
  SELECT
    CAST(NULL AS ARRAY<INT64>) AS value,
    'null_array' as value_type
  UNION ALL
  SELECT
    [NULL] AS value,
    'array_with_nulls' as value_type )
SELECT
    value_type,
    value IS NULL as is_null,
    value IS NOT NULL as is_not_null,
    ARRAY_LENGTH(value) as _array_length,
FROM a
```
Returns:

Row|value_type|is_null|is_not_null|_array_length	
---|---|---|---|---
1|empty_array|false|true|0
2|null_array|true|false|`null`
3|array_with_nulls|false|true|1


## `NULL` and `DATE` operations

As expected, `DATE` operations return `NULL` if any of the input parameters is `NULL`.

```sql
WITH
  dates AS (
  SELECT
    CURRENT_DATE() AS value,
    'current_date' AS value_type
  UNION ALL
  SELECT
    CAST(NULL AS DATE) AS value,
    'null_date' AS value_type)
  SELECT
    value,
    EXTRACT(YEAR FROM value) as year,
    DATE_ADD(value, INTERVAL 7 DAY) as plus_week,
    DATE_SUB(value, INTERVAL 7 DAY) as minus_week,
    DATE_DIFF(value, CURRENT_DATE(), DAY) as day_diff,
    DATE_TRUNC(value, MONTH) as month,
    FORMAT_DATE('%x', value) as us_format,
    LAST_DAY(value, MONTH) as _last_day,
    UNIX_DATE(value) as _unix_date,
  FROM
    dates;
```

Row|value|year|plus_week|minus_week|day_diff|month|us_format|_last_day|_unix_date
---|---|---|---|---|---|---|---|---|---
1|2021-10-05|2021|2021-10-12|2021-09-28|0|2021-10-01|10/05/21|2021-10-31|18905
2|null|null|null|null|null|null|null|null|null

