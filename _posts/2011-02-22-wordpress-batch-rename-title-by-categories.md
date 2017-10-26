---
layout: post
title: WordPress batch rename title by categories

tags: [batch, category, cmd, mysql, post_title, rename, script, sql, taxonomy, title, wordpress, wp]
---

If u need rename many posts by categories, here is sql:

```sql
UPDATE wp_posts
LEFT JOIN wp_term_relationships ON wp_posts.ID = wp_term_relationships.object_id
LEFT JOIN wp_term_taxonomy ON wp_term_relationships.term_taxonomy_id = wp_term_taxonomy.term_taxonomy_id
LEFT JOIN wp_terms ON wp_terms.term_id = wp_term_taxonomy.term_id

SET wp_posts.post_title = CONCAT('Title prefix ', wp_posts.post_title)

WHERE wp_term_taxonomy.taxonomy = 'category'
AND wp_terms.name <> 'Uncategorized'
AND wp_terms.name <> 'My category 1'
AND wp_terms.name <> 'My category 2'
AND wp_posts.post_title NOT LIKE 'Exclude%'
```

First make shure u are going to rename needed scripts with this sql:

```sql
SELECT *

FROM wp_posts
LEFT JOIN wp_term_relationships ON wp_posts.ID = wp_term_relationships.object_id
LEFT JOIN wp_term_taxonomy ON wp_term_relationships.term_taxonomy_id = wp_term_taxonomy.term_taxonomy_id
LEFT JOIN wp_terms ON wp_terms.term_id = wp_term_taxonomy.term_id

WHERE wp_term_taxonomy.taxonomy = 'category'
AND wp_terms.name <> 'Uncategorized'
AND wp_terms.name <> 'My category 1'
AND wp_terms.name <> 'My category 2'
AND wp_posts.post_title NOT LIKE 'Exclude%'
```
