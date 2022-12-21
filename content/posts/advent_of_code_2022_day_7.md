---
layout: post
title: "Advent of Code In BigQuery - Day 7"
author: Peter Boone
tags: ["adventofcode", "tech", "bigquery", "google"]
date: 2022-12-20
draft: false
---

I have fallen behind since day 6, but am attempting to make up some ground. 

Day 7 proved to be pretty tough. I kept track of the "current" directory by appending all of the `cd` commands. I then removed the directory before the `..`, ie `one/dir/..` -> `one`. I attempted to use a recursive regex for this, but unfortunatly BigQuery uses the [re2](https://github.com/google/re2/wiki/Syntax) regex library which does not support recursive syntax. So instead I used a JavaScript UDF for that.

After finding the size of each directory, I then created a CTE with a row for each parent of a directory and the child's size. I then grouped by each of those directories to sum the sizes of all of the children. There might be a better way to do that, but that's the solution I arrived at.

Most of these solutions are ending up as a chain of transformations put into CTEs. 🤷

```sql
CREATE TEMP FUNCTION
  moveUpDir(directory STRING)
  RETURNS STRING
  LANGUAGE js AS r"""
  while (directory.includes("..")) {
    directory = directory.replace(/,\w+,\.\./g, '');
    directory = '/' + directory.replace(/\/(,\.\.)?/g, '');

  }
  return directory;
""";
WITH
  raw_input AS (
  SELECT
    """$ cd /
$ ls
dir a
14848514 b.txt
8504156 c.dat
dir d
$ cd a
$ ls
dir e
29116 f
2557 g
62596 h.lst
$ cd e
$ ls
584 i
$ cd ..
$ cd ..
$ cd d
$ ls
4060174 j
8033020 d.log
5626152 d.ext
7214296 k""" AS raw_text ),
  raw_rows AS (
  SELECT
    o,
    raw_row
  FROM (
    SELECT
      SPLIT(raw_text, '\n') raw_row_array
    FROM
      raw_input ),
    UNNEST(raw_row_array) raw_row
  WITH
  OFFSET
    o ),
  directory_size AS (
  SELECT
    *,
    REGEXP_EXTRACT(raw_row, r'^\d+') AS size,
    REGEXP_EXTRACT(raw_row, r'^\d+ (.*)') AS filename,
    STRING_AGG( REGEXP_EXTRACT(raw_row, r'^\$ cd (.*)'), ',' ) OVER (ORDER BY o) AS current_directory
  FROM
    raw_rows ),
  processed_dir AS (
  SELECT
    DISTINCT moveUpDir(current_directory) AS current_directory,
    raw_row,
    o,
    size,
    filename
  FROM
    directory_size ),
  cleaned_dir AS (
  SELECT
    DISTINCT CONCAT(current_directory, ',', filename) AS filename,
    size,
    current_directory
  FROM
    processed_dir
  WHERE
    filename IS NOT NULL ),
  total_sizes AS (
  SELECT
    SUM(COALESCE(CAST(size AS INT), 0)) AS total_size,
    current_directory
  FROM
    cleaned_dir
  GROUP BY
    current_directory ),
  all_dirs AS (
  SELECT
    total_size,
    SPLIT(current_directory, ',') dirs,
    current_directory
  FROM
    total_sizes ),
  unnest_dirs AS (
  SELECT
    *
  FROM
    all_dirs,
    UNNEST(dirs) d
  WITH
  OFFSET
    o ),
  agg_dirs AS (
  SELECT
    STRING_AGG(d, ',') OVER (PARTITION BY current_directory ORDER BY o) AS dir,
    total_size
  FROM
    unnest_dirs ),
  grouped_dirs AS (
  SELECT
    SUM(total_size) dir_size,
    dir
  FROM
    agg_dirs
  GROUP BY
    dir
  HAVING
    dir_size <= 100000 ),
  part_1 AS (
  SELECT
    SUM(dir_size)
  FROM
    grouped_dirs ),
  all_grouped_dirs AS (
  SELECT
    SUM(total_size) AS dir_size,
    dir
  FROM
    agg_dirs
  GROUP BY
    dir ),
  min_size AS (
  SELECT
    30000000 - (70000000 - MAX(dir_size)) AS size
  FROM
    all_grouped_dirs )
-- SELECT * FROM part_1;
SELECT
  dir_size
FROM
  all_grouped_dirs
JOIN
  min_size
ON
  min_size.size <= all_grouped_dirs.dir_size
ORDER BY
  dir_size
LIMIT
  1 
```