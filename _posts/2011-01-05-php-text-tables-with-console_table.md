---
layout: post
title: php text tables with Console_Table

tags: [console, console_table, formatters, pear, php, table, text, wiki, wysiwyg]
---

http://pear.php.net/package/Console_Table/

класс позволяющий выводить текстовые таблички, то что нужно для консоли

```php
<?php
require_once 'Console/Table.php';

$tbl = new Console_Table();
$tbl->setHeaders(
    array('Language', 'Year')
);
$tbl->addRow(array('PHP', 1994));
$tbl->addRow(array('C',   1970));
$tbl->addRow(array('C++', 1983));

echo $tbl->getTable();

$tbl = new Console_Table();
$tbl->setHeaders(
    array('Language', 'Year')
);
$tbl->addData(
    array(
        array('PHP', 1994),
        array('C',   1970),
        array('C++', 1983)
    )
);
echo $tbl->getTable();
```

оба куска кода выведут вот такую красоту:

```
+----------+------+
| Language | Year |
+----------+------+
| PHP      | 1994 |
| C        | 1970 |
| C++      | 1983 |
+----------+------+
```
