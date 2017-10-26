---
layout: post
title: MySQL many-to-many stored procedure
tags: [mysql, sp]
---

Suppose we have following data:

```
+-------------+----------------+
| keyword     | domain         |
+-------------+----------------+
| foo         | example.com    |
| bar         | example.com    |
| hello world | helloworld.org |
+-------------+----------------+
```

In my case, after normalization, i have following:

```
keywords(id, keyword)
domains(id, domain)
keywords_domains(id, keyword_id, domain_id)
```

`id` in last table is optional and needed for later use.

Here is SQL to create all this:

```sql
-- Drop previously created tables, views and procedures
DROP VIEW IF EXISTS keywords_domains_view;
DROP TABLE IF EXISTS search_results;
DROP TABLE IF EXISTS keywords_domains;
DROP TABLE IF EXISTS keywords;
DROP TABLE IF EXISTS domains;

-- Create tables and views
CREATE TABLE IF NOT EXISTS keywords (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	keyword  VARCHAR(255) NOT NULL UNIQUE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci;

CREATE TABLE IF NOT EXISTS domains (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	domain  VARCHAR(255) NOT NULL UNIQUE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci;

CREATE TABLE IF NOT EXISTS keywords_domains (
	id INT UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	keyword_id INT UNSIGNED NOT NULL,
	domain_id INT UNSIGNED NOT NULL,
	UNIQUE INDEX(keyword_id, domain_id),
	FOREIGN KEY(keyword_id) REFERENCES keywords(id) ON UPDATE CASCADE ON DELETE CASCADE,
	FOREIGN KEY(domain_id) REFERENCES domains(id) ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci;

CREATE VIEW keywords_domains_view AS
	SELECT keyword, domain FROM keywords_domains
	LEFT JOIN keywords ON keywords_domains.keyword_id = keywords.id
	LEFT JOIN domains ON keywords_domains.domain_id = domains.id;
```

Now to if we need to insert some date we will do it something like this:

```sql
INSERT INTO domains VALUES(NULL, 'example.com');
SET @id_1 = LAST_INSERT_ID();

INSERT INTO domains VALUES(NULL, 'helloworld.org');
SET @id_2 = LAST_INSERT_ID();

INSERT INTO keywords VALUES(NULL, 'foo');
INSERT INTO keywords_domains VALUES(NULL, LAST_INSERT_ID(), @id_1);
INSERT INTO keywords VALUES(NULL, 'bar');
INSERT INTO keywords_domains VALUES(NULL, LAST_INSERT_ID(), @id_1);

INSERT INTO keywords VALUES(NULL, 'hello world');
INSERT INTO keywords_domains VALUES(NULL, LAST_INSERT_ID(), @id_2);
```

And all seems ok, except cases when we catch duplicate error.

To make things simpler we can create stored procedure that will check existing of rows and insert them as needed.

Here is stored procedure:

```sql
DELIMITER $$
CREATE PROCEDURE AddKeywordDomain(keyword_in VARCHAR(255), domain_in VARCHAR(255))
BEGIN
	DECLARE keyword_in_id INT DEFAULT 0;
	DECLARE domain_in_id INT DEFAULT 0;
	DECLARE keyword_domain_id INT DEFAULT 0;

	SELECT id INTO domain_in_id FROM domains WHERE domain = domain_in LIMIT 1;
	IF domain_in_id = 0 THEN
		INSERT INTO domains VALUES(NULL, domain_in);
		SET domain_in_id = LAST_INSERT_ID();
	END IF;

	SELECT id INTO keyword_in_id FROM keywords WHERE keyword = keyword_in LIMIT 1;
	IF keyword_in_id = 0 THEN
		INSERT INTO keywords VALUES(NULL, keyword_in);
		SET keyword_in_id = LAST_INSERT_ID();
	END IF;

	SELECT id INTO keyword_domain_id FROM keywords_domains WHERE keyword_id = keyword_in_id AND domain_id = domain_in_id LIMIT 1;
	IF keyword_domain_id = 0 THEN
		INSERT INTO keywords_domains VALUES(NULL, keyword_in_id, domain_in_id);
		SET keyword_domain_id = LAST_INSERT_ID();
	END IF;

	SELECT keyword_domain_id;
END$$
DELIMITER ;
```

And now we can insert new record like this:

```sql
CALL AddKeywordDomain('foo', 'example.com');
CALL AddKeywordDomain('bar', 'example.com');
CALL AddKeywordDomain('hello world', 'helloworld.org');
```
