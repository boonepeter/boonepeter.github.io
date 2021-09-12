---
layout: post
title: "Why does it matter how BigQuery works?"
author: Peter Boone
tags: ["tech", "bigquery", "google"]
date: 2021-09-11
draft: true
---

Google's [Dremel paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36632.pdf) is an interesting paper that explains some of the concepts that underlie BigQuery. I am still processing the paper and have noticed a few things that are relevant to the every day use of BigQuery. I'd recommend starting with [this overview of BigQuery architecture](https://cloud.google.com/blog/products/bigquery/bigquery-under-the-hood) before jumping into the Dremel paper.

## Columnar Data and Records

In BigQuery, data is stored in a columnar format. Instead of storing each row of data 

```sql
-- 817 GB
SELECT *
FROM `bigquery-public-data.github_repos.commits`;

-- 817 GB
SELECT *
FROM `bigquery-public-data.github_repos.commits`
LIMIT 1000;

-- 21.7 GB
SELECT committer
FROM `bigquery-public-data.github_repos.commits`;

-- 12.8 GB
SELECT committer.email
FROM `bigquery-public-data.github_repos.commits`;

-- 612.7 GB
SELECT difference
FROM `bigquery-public-data.github_repos.commits`;

-- 612.7 GB
SELECT diff
FROM `bigquery-public-data.github_repos.commits`,
UNNEST(difference) AS diff;

-- 612.7 GB
SELECT diff
FROM `bigquery-public-data.github_repos.commits`,
UNNEST(difference) AS diff;

-- 1.7 MiB
SELECT diff.old_repo
FROM `bigquery-public-data.github_repos.commits`,
UNNEST(difference) AS diff;
```

## Record Depth

According to the [documentation](https://cloud.google.com/bigquery/docs/nested-repeated#limitations), the maximum depth of nested repeated fields is 15.






## Query Analysis


```sql
-- 476.4 MiB
SELECT
  COUNT(invoice_and_item_number)
FROM
  `bigquery-public-data.iowa_liquor_sales.sales`
WHERE
  zip_code = '52722'
UNION ALL
SELECT
  COUNT(invoice_and_item_number)
FROM
  `bigquery-public-data.iowa_liquor_sales.sales`
WHERE
  zip_code = '50314'
```

#### `S00: Input`
```
READ	
$10:invoice_and_item_number, $11:zip_code
FROM bigquery-public-data.iowa_liquor_sales.sales
WHERE equal($11, '50314')

AGGREGATE
$60 := COUNT($10)

WRITE
$60
TO __stage00_output
```
#### `S01: Input`
```
READ	
$1:invoice_and_item_number, $2:zip_code
FROM bigquery-public-data.iowa_liquor_sales.sales
WHERE equal($2, '52722')

AGGREGATE	
$50 := COUNT($1)

WRITE	
$50
TO __stage01_output
```
#### `S02: Aggregate`
```
READ	
$60
FROM __stage00_output

AGGREGATE	
$40 := SUM_OF_COUNTS($60)

WRITE	
$40
TO __stage02_output
```
#### `S03: Aggregate`
```
READ	
$50
FROM __stage01_output

AGGREGATE	
$30 := SUM_OF_COUNTS($50)

WRITE	
$30
TO __stage03_output
```
### `S04: Output`
```
READ	
$30
FROM __stage03_output

READ	
$40
FROM __stage02_output

WRITE	
$20
TO __stage04_output
```

```sql
-- 476.4 MiB
WITH a AS (
  SELECT
    invoice_and_item_number,
    vendor_name,
    zip_code
  FROM
    `bigquery-public-data.iowa_liquor_sales.sales`
  WHERE
    zip_code IN ('50314', '52722') )
SELECT
  COUNT(invoice_and_item_number)
FROM
  a
WHERE
  zip_code = '52722'
UNION ALL
SELECT
  COUNT(invoice_and_item_number)
FROM
  a
WHERE
  zip_code = '50314';
```

#### `S00: Input`
```
READ	
$10:invoice_and_item_number, $11:zip_code
FROM bigquery-public-data.iowa_liquor_sales.sales
WHERE and(in($11, '50314', '52722'), equal($11, '50314'))

AGGREGATE	
$60 := COUNT($10)

WRITE	
$60
TO __stage00_output
```
#### `S01: Input`
```
READ	
$1:invoice_and_item_number, $2:zip_code
FROM bigquery-public-data.iowa_liquor_sales.sales
WHERE and(in($2, '50314', '52722'), equal($2, '52722'))

AGGREGATE	
$50 := COUNT($1)

WRITE	
$50
TO __stage01_output
```
#### `S02: Aggregate`
```
READ	
$60
FROM __stage00_output

AGGREGATE	
$40 := SUM_OF_COUNTS($60)

WRITE	
$40
TO __stage02_output
```
#### `S03: Aggregate`
```
READ	
$50
FROM __stage01_output

AGGREGATE	
$30 := SUM_OF_COUNTS($50)

WRITE	
$30
TO __stage03_output
```
#### `S04: Output`
```
READ	
$30
FROM __stage03_output

READ	
$40
FROM __stage02_output

WRITE	
$20
TO __stage04_output
```
### Further Reading

- [How BigQuery _actually_ stores the data](https://cloud.google.com/blog/products/bigquery/inside-capacitor-bigquerys-next-generation-columnar-storage-format). Spoiler alert: it's complicated and depends on what your data looks like.
- [How to query columns](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43119.pdf)
- [In memory query execution](https://cloud.google.com/blog/products/bigquery/in-memory-query-execution-in-google-bigquery)