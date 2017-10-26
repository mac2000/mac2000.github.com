---
layout: post
title: reStructuredText

tags: [docutils, markdown, markup, php, rest, restructuredtext, syntax, wiki]
---

Классная штука reStructuredText (reST), очень похоже на Markdown но намного более продвинутей, тут и таблицы и другие полезности.

Спека по разметке:

http://docutils.sourceforge.net/docs/user/rst/quickref.html

Небольшой пример вызова API для преобразования в HTML:

```php
<?php

$rst = 'Hello
=====

mac was here

* list item
* list item

lalla

+------------------------+------------+----------+----------+
| Header row, column 1   | Header 2   | Header 3 | Header 4 |
| (header rows optional) |            |          |          |
+========================+============+==========+==========+
| body row 1, column 1   | column 2   | column 3 | column 4 |
+------------------------+------------+----------+----------+
| body row 2             | Cells may span columns.          |
+------------------------+------------+---------------------+
| body row 3             | Cells may  | - Table cells       |
+------------------------+ span rows. | - contain           |
| body row 4             |            | - body elements.    |
+------------------------+------------+---------------------+
';

$url = 'http://api.rst2a.com/1.0/rst2/html';

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, true);

$data = array(
    'rst' => $rst
);

curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
$output = curl_exec($ch);
$info = curl_getinfo($ch);
curl_close($ch);

echo $output;
```

TODO: проверить наличие php либы, а так же либы для перегона html в reST.

Тул для перегона html2rst:

http://svn.berlios.de/svnroot/repos/docutils/trunk/sandbox/xhtml2rest/xhtml2rest.py

Вся либа написана на Python.
