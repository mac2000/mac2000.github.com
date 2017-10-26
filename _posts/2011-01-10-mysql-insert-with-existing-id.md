---
layout: post
title: MySQL insert with existing id

tags: [database, db, insert, mysql, replace, sql, update]
---

REPLACE INTO - пытается обновить запись, если такова существует, если нет - создает ее.

```sql
REPLACE INTO `transcripts`
SET `ensembl_transcript_id` = 'ENSORGT00000000001',
`transcript_chrom_start` = 12345,
`transcript_chrom_end` = 12678;
```
