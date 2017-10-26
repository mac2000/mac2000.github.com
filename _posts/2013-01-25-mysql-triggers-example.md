---
layout: post
title: MySQL triggers example
tags: [mysql, trigger]
---

Simple example showing how to use triggers in mysql.

Init
----

```sql
DROP TABLE IF EXISTS content;
CREATE TABLE IF NOT EXISTS content(
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	content TEXT NOT NULL
) ENGINE = INNODB DEFAULT CHARSET = utf8 COLLATE = utf8_general_ci;

DROP TABLE IF EXISTS backup;
-- Will store previous version of content
CREATE TABLE IF NOT EXISTS backup(
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	content TEXT NOT NULL
) ENGINE = INNODB DEFAULT CHARSET = utf8 COLLATE = utf8_general_ci;

DROP TABLE IF EXISTS log;
CREATE TABLE IF NOT EXISTS log(
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	message TEXT NOT NULL,
	created TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE = INNODB DEFAULT CHARSET = utf8 COLLATE = utf8_general_ci;


DROP TRIGGER IF EXISTS log_content_added;
DELIMITER $$
CREATE TRIGGER log_content_added AFTER INSERT ON content
FOR EACH ROW
BEGIN
	INSERT INTO log SET message = concat(new.id, ' added');
END$$
DELIMITER ;

DROP TRIGGER IF EXISTS log_content_updated;
DELIMITER $$
CREATE TRIGGER log_content_updated AFTER UPDATE ON content
FOR EACH ROW
BEGIN
	INSERT INTO log SET message = concat(new.id, ' updated');
END$$
DELIMITER ;

DROP TRIGGER IF EXISTS log_content_deleted;
DELIMITER $$
CREATE TRIGGER log_content_deleted AFTER DELETE ON content
FOR EACH ROW
BEGIN
	INSERT INTO log SET message = concat(old.id, ' deleted');

	-- Cleanup backups
	DELETE FROM backup WHERE id = old.id;
END$$
DELIMITER ;

DROP TRIGGER IF EXISTS backup_content;
DELIMITER $$
CREATE TRIGGER backup_content BEFORE UPDATE ON content
FOR EACH ROW
BEGIN
	INSERT INTO backup SET content = old.content;
END$$
DELIMITER ;
```

Tests
-----

```sql
	INSERT INTO content SET content = 'Hello World!';
	ELECT * FROM content;
	+----+--------------+
	| id | content      |
	+----+--------------+
	|  1 | Hello World! |
	+----+--------------+

	SELECT * FROM log;
	+----+---------+---------------------+
	| id | message | created             |
	+----+---------+---------------------+
	|  1 | 1 added | 2013-01-25 11:19:25 |
	+----+---------+---------------------+

	UPDATE content SET content = 'Lorem ipsum' WHERE id = 1;

	SELECT * FROM log;
	+----+-----------+---------------------+
	| id | message   | created             |
	+----+-----------+---------------------+
	|  1 | 1 added   | 2013-01-25 11:44:47 |
	|  2 | 1 updated | 2013-01-25 11:45:21 |
	+----+-----------+---------------------+

	SELECT * FROM backup;
	+----+--------------+
	| id | content      |
	+----+--------------+
	|  1 | Hello World! |
	+----+--------------+

	DELETE FROM content WHERE id = 1;

	SELECT * FROM log;
	+----+-----------+---------------------+
	| id | message   | created             |
	+----+-----------+---------------------+
	|  1 | 1 added   | 2013-01-25 11:44:47 |
	|  2 | 1 updated | 2013-01-25 11:45:21 |
	|  3 | 1 deleted | 2013-01-25 11:46:02 |
	+----+-----------+---------------------+
```