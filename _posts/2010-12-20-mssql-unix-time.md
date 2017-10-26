---
layout: post
title: MsSql Unix time

tags: [mssql, sql, dateadd, datefiff, getdate, timestamp, unixtime, second]
---

```sql
--Convert Unix time 1282234070 to DateTime
SELECT DATEADD(SECOND, 1282234070, {d '1970-01-01'})

--Convert DateTime GETDATE() to Unix time
SELECT DATEDIFF(s, '19700101', GETDATE())
```
