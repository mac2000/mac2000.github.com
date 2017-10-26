---
layout: post
title: MySQL drop all tables in database

tags: [delete, drop, dump, mysql, mysqldump]
---

```sh
mysqldump -u[USERNAME] -p[PASSWORD] --add-drop-table --no-data [DATABASE] | grep ^DROP | mysql -u[USERNAME] -p[PASSWORD] [DATABASE]
```
