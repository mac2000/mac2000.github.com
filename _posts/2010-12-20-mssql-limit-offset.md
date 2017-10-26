---
layout: post
title: MsSQL Limit, Offset

tags: [mssql, sql, limit, offset, rownum, between, row_number, over]
---

```sql
SELECT Paginated.Id, Paginated.Name, Paginated.RowNum
FROM (
    SELECT Branch.Id, Branch.Name, ROW_NUMBER() OVER (ORDER BY Branch.Id) AS RowNum
    FROM Branch
) AS Paginated
WHERE Paginated.RowNum BETWEEN 6 AND 10
```
