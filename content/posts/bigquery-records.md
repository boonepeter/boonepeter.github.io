---
layout: post
title: "Why does it matter how BigQuery works?"
author: Peter Boone
tags: ["tech", "bigquery", "google"]
date: 2021-09-11
draft: true
---

Google's [Dremel paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36632.pdf) is an interesting paper that explains some of the concepts that underlie BigQuery. I am still processing the paper and have noticed a few things that are relevant to the every day use of BigQuery.

```sql

-- 817 GB
-- SELECT *
-- FROM `bigquery-public-data.github_repos.commits`;

-- 817 GB
-- SELECT *
-- FROM `bigquery-public-data.github_repos.commits`
-- LIMIT 1000;

-- 21.7 GB
-- SELECT committer
-- FROM `bigquery-public-data.github_repos.commits`;

-- 12.8 GB
-- SELECT committer.email
-- FROM `bigquery-public-data.github_repos.commits`;

-- 612.7 GB
-- SELECT difference
-- FROM `bigquery-public-data.github_repos.commits`;

-- 612.7 GB
-- SELECT diff
-- FROM `bigquery-public-data.github_repos.commits`,
-- UNNEST(difference) AS diff;

-- 612.7 GB
-- SELECT diff
-- FROM `bigquery-public-data.github_repos.commits`,
-- UNNEST(difference) AS diff;

-- 1.7 MiB
SELECT diff.old_repo
FROM `bigquery-public-data.github_repos.commits`,
UNNEST(difference) AS diff;
```


## Record Depth

According to the [documentation](https://cloud.google.com/bigquery/docs/nested-repeated#limitations), the maximum depth of nested repeated fields is 15.