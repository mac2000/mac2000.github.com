---
layout: post
title: MySql import huge value

tags: [admin, administration, archive, backup, big, database, db, dump, huge, large, mysql]
---

If you getting errors like "MySQL server has gone away" while trying to restore backup, probably you have some very large even huge values in inserts.

To fix this: connect to mysql, set bigger restriction value for `max_allowed_packet` and try import backup again.

```
set global max_allowed_packet=128*1024*1024;
```

Here is how it will look:

```sh
C:\Users\mac>mysql -u root -p[ENTER]
Enter password:[ENTER]
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 414
Server version: 5.5.8 MySQL Community Server (GPL)

Copyright (c) 2000, 2010, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set global max_allowed_packet=128*1024*1024;[ENTER]
Query OK, 0 rows affected (0.03 sec)

mysql> exit[ENTER]
Bye

C:\Users\mac>mysql -u root --password= myhugedb < C:\Users\mac\Desktop\myhugedb.sql[ENTER]
```
