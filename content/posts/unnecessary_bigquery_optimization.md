---
layout: post
title: "Unnecessary BigQuery Optimization"
author: Peter Boone
tags: ["tech", "bigquery", "google"]
date: 2021-09-12
draft: false
---

I was reading [a post](https://cloud.google.com/blog/products/data-analytics/cost-optimization-best-practices-for-bigquery) about BigQuery cost optimization which stated this:

> Also remember you are charged for bytes processed in the [first stage of query execution](https://cloud.google.com/bigquery/query-plan-explanation#background). Avoid creating a complex multistage query just to optimize for bytes processed in the intermediate stages, since there are no cost implications anyway (though you may achieve performance gains).

I was curious about this, because I have written some queries to minimize reading data multiple times. I decided to write two simple queries to test out documentation's advice.

## Query 1 - CTE

```sql
WITH a AS (
  SELECT
    invoice_and_item_number,
    vendor_name,
    zip_code
  FROM `bigquery-public-data.iowa_liquor_sales.sales`
  WHERE zip_code IN ('52722', '50314') )
SELECT
  COUNT(invoice_and_item_number) AS total_sales,
  '52722' AS zip
FROM a
WHERE zip_code = '52722'
UNION ALL
SELECT
  COUNT(invoice_and_item_number) AS total_sales,
  '50314' AS zip
FROM a
WHERE zip_code = '50314';
```

## Query 2 - No CTE
```sql
SELECT
  COUNT(invoice_and_item_number) AS total_sales,
  '52722' AS zip
FROM `bigquery-public-data.iowa_liquor_sales.sales`
WHERE zip_code = '52722'
UNION ALL
SELECT
  COUNT(invoice_and_item_number) AS total_sales,
  '50314' AS zip
FROM `bigquery-public-data.iowa_liquor_sales.sales`
WHERE zip_code = '50314';
```

I would think query 1 would be cheaper since it only reads the data once (in the CTE). Query 2 reads the data twice. But the BigQuery UI says that both queries process the same amount of data (476.4 MiB). I decided to look into the query plans to see how the executions compare.

## Query 1 execution plan

Let's look at the execution plan for query 1. You can focus on `S00` and `S01` since the other stages are exactly the same between the two queries.

```
S00: Input
  READ	
    $10:invoice_and_item_number, $11:zip_code
    FROM bigquery-public-data.iowa_liquor_sales.sales
    WHERE and(in($11, '52722', '50314'), equal($11, '50314'))
  AGGREGATE	
    $80 := COUNT($10)
  WRITE	
    $80
    TO __stage00_output
S01: Input
  READ	
    $1:invoice_and_item_number, $2:zip_code
    FROM bigquery-public-data.iowa_liquor_sales.sales
    WHERE and(in($2, '52722', '50314'), equal($2, '52722'))
  AGGREGATE	
    $70 := COUNT($1)
  WRITE	
    $70
    TO __stage01_output
S02: Aggregate+
  READ	
    $80
    FROM __stage00_output
  COMPUTE	
    $40 := '50314'
  AGGREGATE	
    $60 := SUM_OF_COUNTS($80)
  WRITE	
    $40, $60
    TO __stage02_output
S03: Aggregate+
  READ	
    $70
    FROM __stage01_output
  COMPUTE	
    $30 := '52722'
  AGGREGATE	
    $50 := SUM_OF_COUNTS($70)
  WRITE	
    $30, $50
    TO __stage03_output
S04: Output
  READ	
    $30, $50
    FROM __stage03_output
  READ	
    $40, $60
    FROM __stage02_output
  WRITE	
    $20, $21
    TO __stage04_output
```

## Query 2 execution plan
```
S00: Input
  READ	
    $10:invoice_and_item_number, $11:zip_code
    FROM bigquery-public-data.iowa_liquor_sales.sales
    WHERE equal($11, '50314')
  AGGREGATE	
    $80 := COUNT($10)
  WRITE	
    $80
    TO __stage00_output

S01: Input
  READ	
    $1:invoice_and_item_number, $2:zip_code
    FROM bigquery-public-data.iowa_liquor_sales.sales
    WHERE equal($2, '52722')
  AGGREGATE	
    $70 := COUNT($1)
  WRITE	
    $70
    TO __stage01_output

[... ommiting the rest of the stages]
```

## Comparison

As you can see, both plans read from the public dataset twice. The only difference between the two is in the `WHERE` clauses. The first query has redundant logic:

```sql
WHERE and(in($11, '52722', '50314'), equal($11, '50314'))
```
There is no need to check if the zip code is in `('52722', '50314')` and also equal to `50314`. Compare that to the second, more straightforwad `WHERE` clause:

```sql
WHERE equal($11, '50314')
```

## Query 3

The first two queries prove a point, but a better version of the query would be:

```sql
SELECT
  zip_code,
  COUNT(invoice_and_item_number) AS total_sales,
FROM `bigquery-public-data.iowa_liquor_sales.sales`
WHERE zip_code in ('52722', '50314')
GROUP BY zip_code;
```
The execution plan is more straigntforward and actually only reads the data once. The amount of data processed is the same as the first two queries (476.4 MiB).
```
S00: Input
  READ	
    $1:invoice_and_item_number, $2:zip_code
    FROM bigquery-public-data.iowa_liquor_sales.sales
    WHERE in($2, '52722', '50314')
  AGGREGATE	
    GROUP BY $30 := $2
    $20 := COUNT($1)
  WRITE	
    $30, $20
    TO __stage00_output
    BY HASH($30)
S01: Output
  READ	
    $30, $20
    FROM __stage00_output
  AGGREGATE	
    GROUP BY $40 := $30
    $10 := SUM_OF_COUNTS($20)
  WRITE	
    $40, $10
    TO __stage01_output
```



## Results

Query|Bytes Processed|Slot Time
---|---|---
Query 1|476.4 MiB|8.219 sec
Query 2|476.4 MiB|9.834 sec
Query 3|476.4 MiB|4.761 sec

As the documentation says, we shouldn't worry about the number of times the data is read from a cost perspective. However, there may be performance gains if we can avoid reading the data multiple times. The last query consumes about 50% of the slot time of the first two queries. This is not a thorough comparison, but it does indicate that the third query is much faster.
