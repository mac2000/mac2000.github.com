---
layout: post
title: MySQL import MaxMind World Cities CSV
tags: [mysql, csv, import, maxmind, worldcities]
---

MaxMind has open source/free database of world cities

https://www.maxmind.com/en/worldcities

It is 140+ Mb CSV file containing 3+ M cities

Here is script to import maxmind worldcities into MySQL:

```sql
-- import.sql
DROP TABLE IF EXISTS Cities;
CREATE TABLE Cities (
    Country VARCHAR(2),
    City VARCHAR(100),
    AccentCity VARCHAR(100),
    Region VARCHAR(2),
    Population INT UNSIGNED NULL DEFAULT NULL,
    Latitude FLOAT,
    Longitude FLOAT
);

LOAD DATA INFILE 'C:\\Users\\Alexandr\\Desktop\\worldcitiespop.txt' INTO TABLE Cities
    CHARACTER SET latin1
    FIELDS TERMINATED BY ',' ENCLOSED BY '' LINES TERMINATED BY '\n'
    IGNORE 1 LINES
    (@Country, @City, @AccentCity, @Region, @Population, @Latitude, @Longitude)
SET Country = @Country, City = @City, AccentCity = @AccentCity, Region = @Region,
Population = CASE WHEN @Population = '' THEN NULL ELSE @Population END,
Latitude = @Latitude,
Longitude = @Longitude
;
```


`CHARACTER SET latin1` is needed otherwise you will get `ERROR 1366 (HY000) at line 12: Incorrect string value: '\xE0s' for column 'AccentCity' at row 1`

`IGNORE 1 LINES` is needed to escape headers row, otherwise you will get `ERROR 1406 (22001) at line 12: Data too long for column 'Country' at row 1`

All after `IGNORE 1 LINES` is added just for `Population = CASE WHEN @Population = '' THEN NULL ELSE @Population END` otherwise you will get `ERROR 1366 (HY000) at line 12: Incorrect integer value: '' for column 'Population' at row 1`

And to import it run:

```sh
mysql -u root -proot -e "CREATE DATABASE maxmind";
mysql -u root -proot maxmind < import.sql
```

Here is some numbers:

```sh
C:\>mysql --login-path=local maxmind -e "SELECT COUNT(*) AS Cities FROM Cities"
+---------+
| Cities  |
+---------+
| 3173958 |
+---------+

C:\>mysql --login-path=local maxmind -e "SELECT COUNT(*) AS Countries FROM (SELECT DISTINCT Country FROM Cities) AS q"
+-----------+
| Countries |
+-----------+
|       234 |
+-----------+

C:\>mysql --login-path=local maxmind -e "SELECT COUNT(*) AS Uniq FROM (SELECT DISTINCT Country, City FROM Cities) AS q"
+---------+
| Uniq    |
+---------+
| 2611446 |
+---------+

C:\>mysql --login-path=local maxmind -e "SELECT COUNT(*) AS Population FROM Cities WHERE Population IS NOT NULL"
+------------+
| Population |
+------------+
|      47980 |
+------------+

C:\>mysql --login-path=local maxmind -e "SELECT * FROM Cities WHERE Country = 'ua' AND City IN ('Kiev', 'Kyiv')"\G
*************************** 1. row ***************************
    Country: ua
        City: kiev
AccentCity: Kiev
    Region: 12
Population: 2514227
    Latitude: 50.4333
    Longitude: 30.5167
*************************** 2. row ***************************
    Country: ua
        City: kyiv
AccentCity: Kyiv
    Region: 12
Population: NULL
    Latitude: 50.4333
    Longitude: 30.5167
```


PS: To avoid "Warning: Using a password on the command line interface can be insecure." run:

```sh
mysql_config_editor set --login-path=local --host=localhost --user=root -p
```
