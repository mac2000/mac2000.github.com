---
layout: post
title: Zend application.ini database connection

tags: [application, cfg, conf, config, database, db, ini, mysql, pdo, zend]
---

Here is example for `application.ini`:

```ini
resources.db.adapter = "PDO_MYSQL"
resources.db.params.host = "localhost"
resources.db.params.username = "root"
resources.db.params.password = ""
resources.db.params.dbname = "sakila"
resources.db.isDefaultTableAdapter = true
```

To access default db adapter use:

```php
Zend_Db_Table::getDefaultAdapter()
```
