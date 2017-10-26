---
layout: post
title: MySQL Trigger Validation Example
tags: [mysql, trigger, constraint, check, validation]
---

In sql there is `CHECK` constraint that is ignored by MySQL, so here is workaround with triggers:

```sql
-- SCHEMA
-- ***********************************************************************

DROP TABLE IF EXISTS customer;
CREATE TABLE IF NOT EXISTS customer (
	id INT NOT NULL auto_increment,
	age INT NOT NULL,
	name varchar(128) not null,
	email varchar(128) not null,
	PRIMARY KEY (id),
	UNIQUE KEY email (email)
	-- CONSTRAINT chk_age CHECK (age > 18) -- MySQL ignores constraints
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;



-- STORED PROCEDURES
-- ***********************************************************************

DROP PROCEDURE IF EXISTS validate_customer;
DELIMITER $$
CREATE PROCEDURE validate_customer(
	IN age INT,
	IN email VARCHAR(128)
)
DETERMINISTIC
NO SQL
BEGIN
	IF age < 18 THEN
		SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Age must be gte 18';
	END IF;
	IF NOT (SELECT email REGEXP '$[A-Z0-9._%-]+@[A-Z0-9.-]+\.[A-Z]{2,4}$') THEN
		SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Wrong email';
	END IF;
END$$
DELIMITER ;



-- TRIGGERS
-- ***********************************************************************

DELIMITER $$
CREATE TRIGGER validate_customer_insert
BEFORE INSERT ON customer FOR EACH ROW
BEGIN
	CALL validate_customer(NEW.age, NEW.email);
END$$
DELIMITER ;

DELIMITER $$
CREATE TRIGGER validate_customer_update
BEFORE UPDATE ON customer FOR EACH ROW
BEGIN
	CALL validate_customer(NEW.age, NEW.email);
END$$
DELIMITER ;



-- RUN THEM ALL :)
-- ***********************************************************************

INSERT INTO customer VALUES (NULL, 10, "Alex", "alex@example.com"); -- Error Code: 1644: Age must be gte 18
INSERT INTO customer VALUES (NULL, 20, "Alex", "alex"); -- Error Code: 1644: Wrong email
SELECT * FROM customer; -- Will be empty
```

We declare `validate_customer` stored procedure that will be used in both (insert, update) triggers to check input data.
