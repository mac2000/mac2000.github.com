---
layout: post
title: Fill MySQL calendar table
tags: [mysql, dates]
---

The only possible way to get analysis data from mysql by date ranges with filled missing dates is to have separate calendar in join with results table.

Here is simple procedure that will allow fill calendar table with ranges of dates (supposed that it will be called before analysis):

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

CALL FillCalendar('2013-01-01', '2013-01-03');
CALL FillCalendar('2013-01-01', '2013-01-07');
```
