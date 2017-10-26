---
layout: post
title: MySQL rows to columns

tags: [TODO]
---

Found at: http://www.artfulsoftware.com/infotree/queries.php#78

Sample schema:

![screenshot](/images/wp/18.png)

Script:

```sql
SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='TRADITIONAL';

DROP SCHEMA IF EXISTS `rows2cols` ;
CREATE SCHEMA IF NOT EXISTS `rows2cols` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci ;
USE `rows2cols` ;

-- -----------------------------------------------------
-- Table `rows2cols`.`products`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `rows2cols`.`products` ;

CREATE  TABLE IF NOT EXISTS `rows2cols`.`products` (
    `id` INT NOT NULL AUTO_INCREMENT ,
    `name` VARCHAR(45) NULL ,
    PRIMARY KEY (`id`) )
ENGINE = InnoDB;

-- -----------------------------------------------------
-- Table `rows2cols`.`properties`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `rows2cols`.`properties` ;

CREATE  TABLE IF NOT EXISTS `rows2cols`.`properties` (
    `id` INT NOT NULL ,
    `name` VARCHAR(45) NULL ,
    PRIMARY KEY (`id`) )
ENGINE = InnoDB;

-- -----------------------------------------------------
-- Table `rows2cols`.`product_properties`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `rows2cols`.`product_properties` ;

CREATE  TABLE IF NOT EXISTS `rows2cols`.`product_properties` (
    `id` INT NOT NULL AUTO_INCREMENT ,
    `product_id` INT NOT NULL ,
    `property_id` INT NOT NULL ,
    `value` VARCHAR(45) NULL ,
    PRIMARY KEY (`id`, `product_id`, `property_id`) ,
    INDEX `fk_product_properties_products` (`product_id` ASC) ,
    INDEX `fk_product_properties_properties1` (`property_id` ASC) ,
    CONSTRAINT `fk_product_properties_products`
    FOREIGN KEY (`product_id` )
    REFERENCES `rows2cols`.`products` (`id` )
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
    CONSTRAINT `fk_product_properties_properties1`
    FOREIGN KEY (`property_id` )
    REFERENCES `rows2cols`.`properties` (`id` )
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;

SET SQL_MODE=@OLD_SQL_MODE;
SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS;
SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS;

-- -----------------------------------------------------
-- Data for table `rows2cols`.`products`
-- -----------------------------------------------------
START TRANSACTION;
USE `rows2cols`;
INSERT INTO `rows2cols`.`products` (`id`, `name`) VALUES (1, 'prod1');
INSERT INTO `rows2cols`.`products` (`id`, `name`) VALUES (2, 'prod2');
INSERT INTO `rows2cols`.`products` (`id`, `name`) VALUES (3, 'prod3');

COMMIT;

-- -----------------------------------------------------
-- Data for table `rows2cols`.`properties`
-- -----------------------------------------------------
START TRANSACTION;
USE `rows2cols`;
INSERT INTO `rows2cols`.`properties` (`id`, `name`) VALUES (1, 'price');
INSERT INTO `rows2cols`.`properties` (`id`, `name`) VALUES (2, 'weight');

COMMIT;

-- -----------------------------------------------------
-- Data for table `rows2cols`.`product_properties`
-- -----------------------------------------------------
START TRANSACTION;
USE `rows2cols`;
INSERT INTO `rows2cols`.`product_properties` (`id`, `product_id`, `property_id`, `value`) VALUES (1, 1, 1, '11');
INSERT INTO `rows2cols`.`product_properties` (`id`, `product_id`, `property_id`, `value`) VALUES (2, 1, 2, '22');
INSERT INTO `rows2cols`.`product_properties` (`id`, `product_id`, `property_id`, `value`) VALUES (3, 2, 1, '22');
INSERT INTO `rows2cols`.`product_properties` (`id`, `product_id`, `property_id`, `value`) VALUES (4, 2, 2, '33');
INSERT INTO `rows2cols`.`product_properties` (`id`, `product_id`, `property_id`, `value`) VALUES (5, 3, 1, '33');
INSERT INTO `rows2cols`.`product_properties` (`id`, `product_id`, `property_id`, `value`) VALUES (6, 3, 2, '44');

COMMIT;
```

Here is tables data:

```sh
mysql> use rows2cols;
Database changed
mysql> select * from products;
+----+-------+
| id | name  |
+----+-------+
|  1 | prod1 |
|  2 | prod2 |
|  3 | prod3 |
+----+-------+
3 rows in set (0.00 sec)

mysql> select * from properties;
+----+--------+
| id | name   |
+----+--------+
|  1 | price  |
|  2 | weight |
+----+--------+
2 rows in set (0.00 sec)

mysql> select * from product_properties;
+----+------------+-------------+-------+
| id | product_id | property_id | value |
+----+------------+-------------+-------+
|  1 |          1 |           1 | 11    |
|  2 |          1 |           2 | 22    |
|  3 |          2 |           1 | 22    |
|  4 |          2 |           2 | 33    |
|  5 |          3 |           1 | 33    |
|  6 |          3 |           2 | 44    |
+----+------------+-------------+-------+
6 rows in set (0.00 sec)
```

Bad way to retrive data:

```sh
mysql> SELECT
    ->     products.id,
    ->     products.name,
    ->     properties.name,
    ->     product_properties.value
    ->
    -> FROM products
    -> JOIN product_properties
    -> ON products.id = product_properties.product_id
    -> JOIN properties
    -> ON product_properties.property_id = properties.id;
+----+-------+--------+-------+
| id | name  | name   | value |
+----+-------+--------+-------+
|  1 | prod1 | price  | 11    |
|  1 | prod1 | weight | 22    |
|  2 | prod2 | price  | 22    |
|  2 | prod2 | weight | 33    |
|  3 | prod3 | price  | 33    |
|  3 | prod3 | weight | 44    |
+----+-------+--------+-------+
6 rows in set (0.00 sec)
```

Good way:

```sh
mysql> SELECT
    ->     products.id,
    ->     products.name,
    ->     GROUP_CONCAT(if(properties.name = 'price', value, NULL)) AS 'price',
    ->     GROUP_CONCAT(if(properties.name = 'weight', value, NULL)) AS 'weight'

    ->
    -> FROM products
    -> JOIN product_properties
    -> ON products.id = product_properties.product_id
    -> JOIN properties
    -> ON product_properties.property_id = properties.id
    -> GROUP BY products.id;
+----+-------+-------+--------+
| id | name  | price | weight |
+----+-------+-------+--------+
|  1 | prod1 | 11    | 22     |
|  2 | prod2 | 22    | 33     |
|  3 | prod3 | 33    | 44     |
+----+-------+-------+--------+
3 rows in set (0.00 sec)
```

**So here is SQL:**

```sql
SELECT
    products.id,
    products.name,
    GROUP_CONCAT(if(properties.name = 'price', value, NULL)) AS 'price',
    GROUP_CONCAT(if(properties.name = 'weight', value, NULL)) AS 'weight'

FROM products
JOIN product_properties
ON products.id = product_properties.product_id
JOIN properties
ON product_properties.property_id = properties.id
GROUP BY products.id
```
