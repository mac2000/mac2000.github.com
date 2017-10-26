---
layout: post
title: MySQL выборка с использованием parent_id

tags: [mysql, sql, join, left, create, table, primary]
---

Простенькая задача: Существует таблица содержащая поля `id`, `title` и `parent_id`.  Известно что данные могут быть вложены друг в друга не более чем на два уровня. Необходимо вывести список “детей”, `title`, которых будет содержать как свой заголовок, так и заголовок родителя.Для начала пример базы:

```sql
CREATE TABLE IF NOT EXISTS `tree` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `title` varchar(200) NOT NULL,
    `parent_id` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`)
);

INSERT INTO `tree` (`id`, `title`, `parent_id`) VALUES
(1, 'parent1', NULL),
(2, 'parent2', NULL),
(3, 'parent3', NULL),
(4, 'child1', 1),
(5, 'child2', 1),
(6, 'child3', 3);
```

и собственно сам запрос:

```sql
SELECT child.id, CONCAT(parent.title, ' - ', child.title) AS title
FROM tree AS child
LEFT JOIN tree As parent
ON child.parent_id = parent.id
WHERE child.parent_id IS NOT NULL
ORDER BY title;
```

Результатом работы которого будет:

```
+----+------------------+
| id | title            |
+----+------------------+
|  4 | parent1 - child1 |
|  5 | parent1 - child2 |
|  6 | parent3 - child3 |
+----+------------------+
```
