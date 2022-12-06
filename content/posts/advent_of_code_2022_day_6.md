---
layout: post
title: "Advent of Code In BigQuery - Day 6"
author: Peter Boone
tags: ["adventofcode", "tech", "bigquery", "google"]
date: 2022-12-06
draft: false
---

Okay, hope in SQL has been temporarily restored. This problem fits SQL a bit better so I did not have to do anything too wild. For part 2, I decided to copy/past LAG(4) - LAG(13). There might be a better way to do it but I decided to keep it simple for myself.

```sql
WITH raw_input AS (
  SELECT
   'mjqjpqmgbljsphdztnvjfqwrcgsmlb' AS raw_row
),
points as (
  SELECT
    TO_CODE_POINTS(raw_row) p
  FROM raw_input
),
point_rows AS (    
  SELECT 
    CHR(cp) cp, o
  FROM points,
  UNNEST(p) cp WITH offset o
),
points_grouped AS (
  SELECT
    CONCAT(
      LAG(cp, 1) OVER (ORDER BY o),
      LAG(cp, 2) OVER (ORDER BY o),
      LAG(cp, 3) OVER (ORDER BY o),
      LAG(cp, 4) OVER (ORDER BY o),
      LAG(cp, 5) OVER (ORDER BY o),
      LAG(cp, 6) OVER (ORDER BY o),
      LAG(cp, 7) OVER (ORDER BY o),
      LAG(cp, 8) OVER (ORDER BY o),
      LAG(cp, 9) OVER (ORDER BY o),
      LAG(cp, 10) OVER (ORDER BY o),
      LAG(cp, 11) OVER (ORDER BY o),
      LAG(cp, 12) OVER (ORDER BY o),
      LAG(cp, 13) OVER (ORDER BY o),
      cp
    ) str,
    o
  FROM point_rows
)
SELECT o + 1
FROM points_grouped
WHERE ARRAY_LENGTH(ARRAY((
  SELECT DISTINCT element FROM UNNEST(SPLIT(str,'')) as element
))) = 14
ORDER BY o
LIMIT 1
```