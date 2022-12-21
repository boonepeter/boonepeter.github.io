---
layout: post
title: "Advent of Code In BigQuery - Day 8"
author: Peter Boone
tags: ["adventofcode", "tech", "bigquery", "google"]
date: 2022-12-21
draft: false
---

I enjoyed this one! Windowing functions worked nicely for this.

```sql
WITH
  raw_input AS (
  SELECT
    """30373
25512
65332
33549
35390""" AS raw_text ),
raw_rows AS (
  SELECT
    o,
    raw_row
  FROM (
    SELECT
      SPLIT(raw_text, '\n') raw_row_array
    FROM
      raw_input ),
    UNNEST(raw_row_array) raw_row WITH OFFSET o
),
split_rows AS (
  SELECT SPLIT(raw_row, '') as split_row,
  raw_row,
  o
  FROM raw_rows
), unnested_rows AS (
  SELECT
    raw_row,
    o AS row_num,
    col_num,
    CAST(col AS INT) AS col
  FROM split_rows,
  UNNEST(split_row) col WITH offset col_num
), agg_rows as (
  SELECT
    ARRAY_AGG(col) OVER (PARTITION BY col_num ORDER BY row_num ASC) AS down,
    ARRAY_AGG(col) OVER (PARTITION BY col_num ORDER BY row_num DESC) AS up,
    ARRAY_AGG(col) OVER (PARTITION BY row_num ORDER BY col_num ASC) AS left_,
    ARRAY_AGG(col) OVER (PARTITION BY row_num ORDER BY col_num DESC) AS right_,
    col,
    col_num,
    row_num,
    raw_row
  FROM unnested_rows
), part_1 AS (
  SELECT
    COUNT(1)
  FROM agg_rows
  WHERE
    -- if evaluates to null, is an edge tree
    IFNULL(
      col > (SELECT MAX(down[SAFE_OFFSET(o - 1)]) FROM UNNEST(down) x WITH OFFSET o) -- visible down
      OR col > (SELECT MAX(up[SAFE_OFFSET(o - 1)]) FROM UNNEST(up) x WITH OFFSET o) -- visible up,
      OR col > (SELECT MAX(left_[SAFE_OFFSET(o - 1)]) FROM UNNEST(left_) x WITH OFFSET o) -- visible left,
      OR col > (SELECT MAX(right_[SAFE_OFFSET(o - 1)]) FROM UNNEST(right_) x WITH OFFSET o) -- visible right,
    , TRUE)
), max_counts as (
  SELECT
    MAX(row_num) max_row,
    MAX(col_num) max_col
  FROM agg_rows
), line_of_sight AS (
  SELECT
    row_num - IFNULL((SELECT MAX(o) FROM UNNEST(down) x WITH OFFSET o WHERE x >= col AND o <> row_num), 0) AS down_visible,
    (SELECT MAX(o) FROM UNNEST(up) x WITH OFFSET o WHERE x >= col AND o <> (max_row - row_num)) as up,
    (max_row - row_num) - IFNULL((SELECT MAX(o) FROM UNNEST(up) x WITH OFFSET o WHERE x >= col AND o <> (max_row - row_num)), 0) AS up_visible,
    col_num - IFNULL((SELECT MAX(o) FROM UNNEST(left_) x WITH OFFSET o WHERE x >= col AND o <> col_num), 0) AS left_visible,
    (max_col - col_num) - IFNULL((SELECT MAX(o) FROM UNNEST(right_) x WITH OFFSET o WHERE x >= col AND o <> (max_col - col_num)), 0) AS right_visible,
    *
  FROM agg_rows, max_counts
)
-- SELECT * FROM part_1
SELECT MAX(down_visible * up_visible * left_visible * right_visible) AS max_scenic_score
FROM line_of_sight
```
