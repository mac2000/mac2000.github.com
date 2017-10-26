---
layout: post
title: Modx Anithing rating &#038; menuindex

tags: [anithingrating, modx, mysql, php]
---

В моем случае необходимо подправлять menuindex в зависимости от того какой рейтинг стоит у документа:

```sql
UPDATE modx_site_content AS cnt
LEFT JOIN (
        SELECT
        t1.id,
        IF(CAST(IFNULL(t2.value,0) AS SIGNED) = 1,1,
            ABS((CAST(IFNULL(t3.rating,0) AS SIGNED)-99))
            ) AS menuindex
        FROM modx_site_content as t1
        LEFT JOIN modx_site_tmplvar_contentvalues as t2
        ON t1.id = t2.contentid AND t2.tmplvarid = 31
        LEFT JOIN modx_atRating_grpsalon as t3
        ON t3.rating_id = t1.id
        WHERE t1.template = 7
        ORDER BY menuindex ASC, id ASC
        ) AS val
ON cnt.id = val.id
SET cnt.menuindex = val.menuindex
```
