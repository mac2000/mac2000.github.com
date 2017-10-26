---
layout: post
title: MySql profiling

tags: [admin, db, log, mysql, performance, sql, webgrind, php, xdebug, profiler, general_log, slow_query_log]
---

http://code.google.com/p/webgrind/wiki/Installation

http://anantgarg.com/2009/03/10/php-xdebug-webgrind-installation/

* Put webgrind into `C:\xampp\htdocs`
* Create `C:\xampp\htdocs\webgrind\tmp` folder and allow full access to it for everyone
* Modify `C:\xampp\php\php.ini` file:

```ini
xdebug.profiler_output_dir = "C:\xampp\htdocs\webgrind\tmp"
xdebug.profiler_output_name = "cachegrind.out.%t.%p"
xdebug.profiler_enable = 1
```

* Restart apache
* Open page that you want to profile, then open `http://localhost/webgrind`

MySQL
-----

For mysql logs change `C:\xampp\mysql\bin\my.ini`

```ini
general_log = 1
slow_query_log = 1
```

Logs will be written to

```
C:/xampp/mysql/data/mysql.log
C:/xampp/mysql/data/mysql-slow.log
```
