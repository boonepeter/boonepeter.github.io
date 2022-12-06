---
layout: post
title: "Advent of Code In BigQuery"
author: Peter Boone
tags: ["tech", "bigquery", "google"]
date: 2022-12-04
draft: false
---

This year, I am be taking a page out of [Felipe Hoffa's book](https://towardsdatascience.com/advent-of-code-sql-bigquery-31e6a04964d4) and will be attempting to complete the Advent of Code using BigQuery SQL. Until I reach the point where I can't solve the puzzles with SQL, I'll keep track of things here.

## Day 1

```sql
WITH
  raw_input AS (
  /*
    To start, I just copy/paste the input as 
    a raw text string, and then parse that 
    string into rows.

    This feels true to the spirit of the challenge. I'll reuse this parsing on future days.
  */
  SELECT
    """1000
2000
3000

4000

5000
6000

7000
8000
9000

10000""" AS raw_text ),
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
elf_ids AS (
  -- Rolling count of emtpy rows as the elf id
  SELECT
    COUNTIF(raw_row = '') OVER (ORDER BY o) AS elf_id,
    o,
    CAST(IF(raw_row = '', '0', raw_row) AS INT) AS calories
  FROM raw_rows
),
part_1 AS (
  SELECT
    SUM(calories),
    elf_id
  FROM
    elf_ids
  GROUP BY elf_id
  ORDER BY SUM(calories) DESC
  LIMIT 1
),
top_elves AS (
SELECT
  SUM(calories) calorie_sum,
  elf_id
FROM
  elf_ids
GROUP BY elf_id
ORDER BY SUM(calories) DESC
LIMIT 3
)
-- Part 2
SELECT
  SUM(calorie_sum) total_calories
FROM top_elves;
```

## Day 2

```sql
WITH
  raw_input AS (
  SELECT
    """A Y
B X
C Z""" AS raw_text ),
  raw_rows AS (
    SELECT
      o,
      -- A = 65, X = 88
      -- Subtract on less than that to get the
      -- 1-indexed value of rock/paper/scissors
      TO_CODE_POINTS(raw_row)[OFFSET(0)] - 64 AS opponent,
      TO_CODE_POINTS(raw_row)[OFFSET(2)] - 87 AS me,
      raw_row
    FROM (
      SELECT
        SPLIT(raw_text, '\n') raw_row_array
      FROM
        raw_input ),
      UNNEST(raw_row_array) raw_row WITH OFFSET o
  ),

  round_results_1 AS (
    SELECT
      CASE
        -- tie
        WHEN opponent = me THEN 3
        -- win. 3 - 2, 2 - 1, or 1 - 3
        WHEN me - opponent IN (1, -2) THEN 6
      ELSE 0
    END AS my_result,
      *
    FROM
      raw_rows ),

  part_1 AS (
    SELECT
      SUM(me + my_result)
    FROM
      round_results_1 ),

  round_results_2 AS (
    SELECT
      CASE
        -- "me" is the expected result
        -- tie
        WHEN me = 2 THEN opponent
        -- 3 -> 1, 2 -> 3, 1 -> 2
        WHEN me = 3 THEN MOD(opponent, 3) + 1
        -- 1 -> 3, 2 -> 1, 3 -> 2
        WHEN me = 1 THEN MOD(opponent + 4, 3) + 1
      ELSE 0
    END AS my_pick,
      *
    FROM
      raw_rows ),

  part_2 AS (
    SELECT
      SUM((me - 1) * 3 + my_pick)
    FROM
      round_results_2 )
SELECT
  *
FROM
  part_2;
```

## Day 3

```sql
WITH
  raw_input AS (
  SELECT
    """vJrwpWtwJgWrhcsFMMfFFhFp
jqHRNqRjqzjGDLGLrsFMfFZSrLrFZsSL
PmmdzqPrVvPwwTWBwg
wMqvLMZHhHMvwLHjbvcjnnSBnvTQFn
ttgJtRGJQctTZtZT
CrZsJsPPZsGzwwsLwLmpwMDw""" AS raw_text ),
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
common_letters as (
-- substring into two halves, split into characters, get the intersection,
-- convert to code points, subtract 64 ('A' is 65)
  SELECT
    *,
    TO_CODE_POINTS((
      (SELECT * FROM UNNEST(SPLIT(SUBSTR(raw_row, 0, DIV(LENGTH(raw_row), 2)), '')))
        INTERSECT DISTINCT
      (SELECT * FROM UNNEST(SPLIT(SUBSTR(raw_row, DIV(LENGTH(raw_row), 2) + 1), '')))
    ))[OFFSET(0)] - 64 as code_point,
  FROM raw_rows
),
part_1 AS (
  SELECT
    SUM(IF(code_point < 27, code_point + 26, code_point - 32))
  FROM common_letters
),
group_of_three as (
  SELECT 
    LAG(raw_row, 2) OVER (ORDER BY o) as lag_2,
    LAG(raw_row) OVER (ORDER BY o) as lag_1,
    *,
  FROM raw_rows
),
common_three AS (
  -- get intersection of groups of 3 rows. Probably could be done in one pass...
  SELECT
    TO_CODE_POINTS((
      (SELECT * FROM UNNEST(SPLIT(lag_2, '')))
      INTERSECT DISTINCT
      (SELECT * FROM UNNEST(SPLIT(lag_1, '')))
      INTERSECT DISTINCT
      (SELECT * FROM UNNEST(SPLIT(raw_row, '')))
    ))[OFFSET(0)] - 64 AS code_point,
    *
  FROM group_of_three
  WHERE MOD(o, 3) = 2
),
part_2 as (
  SELECT
    SUM(IF(code_point < 27, code_point + 26, code_point - 32))
  FROM common_three
)
SELECT *
FROM part_2;
```

## Day 4

```sql
WITH
  raw_input AS (
  SELECT
    """2-4,6-8
2-3,4-5
5-7,7-9
2-8,3-7
6-6,4-6
2-6,4-8""" AS raw_text ),
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
parsed_rows AS (
  -- Get digits and cast to INT. They are in the expected order
  SELECT
    ARRAY(
      SELECT
        CAST(n AS INT)
      FROM
      UNNEST(
        REGEXP_EXTRACT_ALL(raw_row, r'\d+')) n
    ) AS sections,
    *
  FROM raw_rows
),
part_1 as (
  -- could use BETWEEN here...but I like to avoid SQL when possible (lol).
  SELECT
    (
      sections[OFFSET(0)] >= sections[OFFSET(2)]
      AND sections[OFFSET(1)] <= sections[OFFSET(3)]
    )
    OR (
      sections[OFFSET(2)] >= sections[OFFSET(0)]
      AND sections[OFFSET(3)] <= sections[OFFSET(1)]
    ) as overlap,
    *
  FROM parsed_rows
),
part_2 AS (
  SELECT
    (
      sections[OFFSET(1)] >= sections[OFFSET(2)]
      AND sections[OFFSET(0)] <= sections[OFFSET(3)]
    )
    AS overlap,
    *
  FROM parsed_rows
)
SELECT 
  COUNTIF(overlap)
FROM part_2;
```

## Day 5

[Day 5](/posts/advent_of_code_2022_day_5)