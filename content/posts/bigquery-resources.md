---
layout: post
title: "BigQuery Resources"
author: Peter Boone
date: 2022-03-01
tags: ["bigquery", "tech"]
draft: false
---

This is a living document of BigQuery resources that I have found useful (with some comments about their usefulness).

## "Under the hood" articles

[Anatomy of a BigQuery Query](https://cloud.google.com/blog/products/bigquery/anatomy-of-a-bigquery-query): Why is BigQuery cool?

[BigQuery under the hood](https://cloud.google.com/blog/products/bigquery/bigquery-under-the-hood): High-level descriptioon of how BigQuery is able to be so fast.

[In memory query execution](https://cloud.google.com/blog/products/bigquery/in-memory-query-execution-in-google-bigquery): If you like map reduce you should read this article. It talks about what is unique about BigQuery's shuffle.

[Dremel](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36632.pdf): The paper about Dremel (Google's internal name for BigQuery). If you're interested in data structures you should check out this paper and learn about repetition and definition levels.

[How does Google compress the columns](https://cloud.google.com/blog/products/bigquery/inside-capacitor-bigquerys-next-generation-columnar-storage-format) (it's complicated and depends on what your data looks like)

## Performance

[IO best practices](https://cloud.google.com/bigquery/docs/best-practices-performance-input): Mainly, avoid `SELECT *` queries.

[Communication performance](https://cloud.google.com/bigquery/docs/best-practices-performance-communication): reduce datta before joins, use WITH clauses primarily for readability (they aren't materialized)

[Optimize query computation](https://cloud.google.com/bigquery/docs/best-practices-performance-compute): Practical tips for how to make BigQuery queries faster.

[SQL anti-patterns](https://cloud.google.com/bigquery/docs/best-practices-performance-patterns): A list of common SQL anti-patterns to avoid.

## Style

[dbt style guide](https://github.com/dbt-labs/corp/blob/main/dbt_style_guide.md): Some good general advice that is aplicable to BigQuery.

[Gitlab style guide](https://about.gitlab.com/handbook/business-technology/data-team/platform/sql-style-guide/)

## My articles

[BigQuery records](/posts/bigquery-records)

[Null in BigQuery](/posts/null-in-bigquery)

[Unnecessary BigQuery optimization](/posts/unnecessary_bigquery_optimization)
