---
layout: post
title: MySQL payment table analysis
tags: [mysql, analysis, olap, cube, dates]
---

http://downloads.mysql.com/docs/sakila-db.zip - training database with payment table.

Run following to install it:

```sh
mysql -u root -p < sakila-schema.sql
mysql -u root -p < sakila-data.sql
```

Here is how our payment table fields:

```sql
SELECT COLUMN_NAME FROM information_schema.columns WHERE TABLE_NAME = 'payment' AND TABLE_SCHEMA = 'sakila';
```

```
+--------------+
| COLUMN_NAME  |
+--------------+
| payment_id   |
| customer_id  |
| staff_id     |
| rental_id    |
| amount       |
| payment_date |
| last_update  |
+--------------+
```

We are interested in staff_id, amount and payment_date and want to get some analysis data on them.

ROLLUP
------

Usually we doing something like this:

```sql
SELECT staff_id, SUM(amount) AS total FROM sakila.payment GROUP BY staff_id;
```

```
+----------+----------+
| staff_id | total    |
+----------+----------+
|        1 | 33489.47 |
|        2 | 33927.04 |
+----------+----------+
```

And here is what `ROLLUP` can do:

```sql
SELECT staff_id, SUM(amount) AS total FROM sakila.payment GROUP BY staff_id WITH ROLLUP;
```

```
+----------+----------+
| staff_id | total    |
+----------+----------+
|        1 | 33489.47 |
|        2 | 33927.04 |
|     NULL | 67416.51 |
+----------+----------+
```

So rollup gathers information and counting total, it can be used to build cubes.

Here is more complicated example:

```sql
SELECT
	YEAR(payment_date) AS payment_year,
	MONTH(payment_date) AS payment_month,
	SUM(amount) AS total
FROM sakila.payment
GROUP BY
	YEAR(payment_date),
	MONTH(payment_date)
	WITH ROLLUP;
```

```
+--------------+---------------+----------+
| payment_year | payment_month | total    |
+--------------+---------------+----------+
|         2005 |             5 |  4824.43 |
|         2005 |             6 |  9631.88 |
|         2005 |             7 | 28373.89 |
|         2005 |             8 | 24072.13 |
|         2005 |          NULL | 66902.33 |
|         2006 |             2 |   514.18 |
|         2006 |          NULL |   514.18 |
|         NULL |          NULL | 67416.51 |
+--------------+---------------+----------+
```

Notice how rollup counts sums for 2005, 2006 and total.

Date ranges
-----------

Lets try this:

```sql
SELECT
	DATE(payment_date) AS payment_date,
	SUM(amount) AS total
FROM sakila.payment
WHERE DATE(payment_date) BETWEEN '2005-08-01' AND '2005-08-31'
GROUP BY DATE(payment_date);
```

```
+--------------+---------+
| payment_date | total   |
+--------------+---------+
| 2005-08-01   | 2817.29 |
| 2005-08-02   | 2726.57 |
| 2005-08-16   |  111.77 |
| 2005-08-17   | 2457.07 |
| 2005-08-18   | 2710.79 |
| 2005-08-19   | 2615.72 |
| 2005-08-20   | 2723.76 |
| 2005-08-21   | 2809.41 |
| 2005-08-22   | 2576.74 |
| 2005-08-23   | 2523.01 |
+--------------+---------+
```

You can see that we have no payments in the first half of the month.

But lets assume that we need this data with zeros, to build some charts in our reports.


After some searching seems that only possible way to do this in MySQL is to create separate calendar table and join in with results table.

So, here is SQL to create table and stored procedure:

```sql
DROP PROCEDURE IF EXISTS FillCalendar;
DROP TABLE IF EXISTS calendar;

CREATE TABLE IF NOT EXISTS calendar(
	calendar_date DATE NOT NULL PRIMARY KEY
);

DELIMITER $$
CREATE PROCEDURE FillCalendar(start_date DATE, end_date DATE)
BEGIN
	DECLARE crt_date DATE;
	SET crt_date = start_date;
	WHILE crt_date <= end_date DO
		INSERT IGNORE INTO calendar VALUES(crt_date);
		SET crt_date = ADDDATE(crt_date, INTERVAL 1 DAY);
	END WHILE;
END$$
DELIMITER ;
```

And now we can call our procedure to fill desired dates range like this:

```sql
CALL FillCalendar('2005-08-01', '2005-08-31');
```

Now lets try to make query joining our calendar and payment tables:

```sql
SELECT
	calendar.calendar_date AS payment_date,
	IFNULL(SUM(payment.amount), 0) AS total
FROM calendar
LEFT JOIN payment ON calendar.calendar_date = DATE(payment.payment_date)
WHERE calendar.calendar_date BETWEEN '2005-08-01' AND '2005-08-31'
GROUP BY calendar.calendar_date;
```

```
+--------------+---------+
| payment_date | total   |
+--------------+---------+
| 2005-08-01   | 2817.29 |
| 2005-08-02   | 2726.57 |
| 2005-08-03   |    0.00 |
| 2005-08-04   |    0.00 |
| 2005-08-05   |    0.00 |
| 2005-08-06   |    0.00 |
| 2005-08-07   |    0.00 |
| 2005-08-08   |    0.00 |
| 2005-08-09   |    0.00 |
| 2005-08-10   |    0.00 |
| 2005-08-11   |    0.00 |
| 2005-08-12   |    0.00 |
| 2005-08-13   |    0.00 |
| 2005-08-14   |    0.00 |
| 2005-08-15   |    0.00 |
| 2005-08-16   |  111.77 |
| 2005-08-17   | 2457.07 |
| 2005-08-18   | 2710.79 |
| 2005-08-19   | 2615.72 |
| 2005-08-20   | 2723.76 |
| 2005-08-21   | 2809.41 |
| 2005-08-22   | 2576.74 |
| 2005-08-23   | 2523.01 |
| 2005-08-24   |    0.00 |
| 2005-08-25   |    0.00 |
| 2005-08-26   |    0.00 |
| 2005-08-27   |    0.00 |
| 2005-08-28   |    0.00 |
| 2005-08-29   |    0.00 |
| 2005-08-30   |    0.00 |
| 2005-08-31   |    0.00 |
+--------------+---------+
```

Mission is almost accomplished, now we can split report into years, monthes, staffs and make rollups on them.

```sql
SELECT
	YEAR(calendar.calendar_date) AS payment_year,
	MONTH(calendar.calendar_date) AS payment_month,
	IFNULL(SUM(payment.amount), 0) AS total
FROM calendar
LEFT JOIN payment ON calendar.calendar_date = DATE(payment.payment_date)
WHERE calendar.calendar_date BETWEEN '2005-07-01' AND '2006-03-01'
GROUP BY YEAR(calendar.calendar_date), MONTH(calendar.calendar_date) WITH ROLLUP;
```

```
+--------------+---------------+----------+
| payment_year | payment_month | total    |
+--------------+---------------+----------+
|         2005 |             7 | 28373.89 |
|         2005 |             8 | 24072.13 |
|         2005 |             9 |     0.00 |
|         2005 |            10 |     0.00 |
|         2005 |            11 |     0.00 |
|         2005 |            12 |     0.00 |
|         2005 |          NULL | 52446.02 |
|         2006 |             1 |     0.00 |
|         2006 |             2 |   514.18 |
|         2006 |             3 |     0.00 |
|         2006 |          NULL |   514.18 |
|         NULL |          NULL | 52960.20 |
+--------------+---------------+----------+
```

Yes it is cube :)