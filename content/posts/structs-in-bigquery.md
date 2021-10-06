---
layout: post
title: "Comparing STRUCT in BigQuery"
author: Peter Boone
tags: ["tech", "bigquery", "google"]
date: 2021-10-5
draft: true
---

Struct comparison in BigQuery is a [bit tricky](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types#:~:text=Notice%2C%20though%2C%20that%20these%20direct%20equality%20comparisons%20compare%20the%20fields%20of%20the%20STRUCT%20pairwise%20in%20ordinal%20order%20ignoring%20any%20field%20names.%20If%20instead%20you%20want%20to%20compare%20identically%20named%20fields%20of%20a%20STRUCT%2C%20you%20can%20compare%20the%20individual%20fields%20directly.).

When comparing two STRUCTs, the __order__ of the fields is used, not the __field names__.

For example:

```sql
 SELECT
    STRUCT('a' AS a, 'b' AS b) = STRUCT('a' AS b, 'b' AS a), -- true
    STRUCT('a' AS a, 'b' AS b) = STRUCT('b' AS b, 'a' AS a), -- false
    STRUCT('a' AS a, 'b' AS b) = STRUCT('a' AS aaaaaaaa, 'b' AS bbbbbbbbbbbb), -- true
```