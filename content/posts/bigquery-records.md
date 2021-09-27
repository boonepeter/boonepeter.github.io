---
layout: post
title: "The practical use of repetition and definition levels in BigQuery"
author: Peter Boone
tags: ["tech", "bigquery", "google"]
date: 2021-09-12
draft: false
---

Google's [Dremel paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36632.pdf) is an interesting read that explains some of the concepts that underlie BigQuery. I am still processing the paper and have noticed a few things about repetition and definition levels that are relevant to the every day use of BigQuery.

## Columnar Data and Records

The underlying storage format for BigQuery is columnar. One of the first pieces of advice given to people using BigQuery is to only select the rows that you need.

To demonstrate, this query will process __817 GB__ of data:
```sql
SELECT *
FROM `bigquery-public-data.github_repos.commits`;
```

Limiting the number of rows does not reduce the amount of data that is processed (this is still __817 GB__):
```sql
SELECT *
FROM `bigquery-public-data.github_repos.commits`
LIMIT 1000;
```

In contrast, if you select `commiier` from this dataset, only __21.7 GB__ is processed:
```sql
SELECT committer
FROM `bigquery-public-data.github_repos.commits`;
```

This also applies to `RECORD` fields. This only processes __12.8 GB__:
```sql
SELECT committer.email
FROM `bigquery-public-data.github_repos.commits`;
```

When reading about the repetition and definition levels in BigQuery's data format, I realized that limiting the selection also applies to repeated fields.

In this dataset `difference` is an array. This query processes __612.7 GB__:

```sql
SELECT diff
FROM `bigquery-public-data.github_repos.commits`,
UNNEST(difference) AS diff;
```

The real breakthrough I had was realizing that you can just select a field from the repeated record. This query only processes __1.7 MiB__:

```sql
SELECT diff.old_repo
FROM `bigquery-public-data.github_repos.commits`,
UNNEST(difference) AS diff;
```

This could greatly reduce the amount of data you have to process if you are only interested in certain fields of the repeated record. In the above example, by selecting only the `old_repo` field we process 0.00027777777% of the data that we would have by selecting the entire record.

## Record Depth

According to the [documentation](https://cloud.google.com/bigquery/docs/nested-repeated#limitations), the maximum depth of nested repeated fields is 15. I don't think many people will come up against that limit. Just to double check, this is what happens when you try to create a schema that has too many levels.

![Nested Record Schema](/imgs/bigquery-records/nested_record_schema.png)

The error message:
![Error](/imgs/bigquery-records/error.png)

The Dremel paper says:

> Levels are packed as bit sequences. We
only use as many bits as necessary; for example, if the maximum
definition level is 3, we use 2 bits per definition level.

I'm assuming the maximum definition level of 15 is to limit this information to a single byte (4 bits for repetition level and 4 bits for definition level).

### Further Reading

- [How BigQuery _actually_ stores the data](https://cloud.google.com/blog/products/bigquery/inside-capacitor-bigquerys-next-generation-columnar-storage-format). Spoiler alert: it's complicated and depends on what your data looks like.