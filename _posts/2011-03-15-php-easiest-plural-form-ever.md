---
layout: post
title: php easiest plural form ever

tags: [form, php, plural]
---

```php
<?php

function plural_form($number, $after) {
    $cases = array (2, 0, 1, 1, 1, 2);
    return $number.' '.$after[ ($number%100>4 && $number%100<20)? 2: $cases[min($number%10, 5)] ];
}

$c = array('комментарий','комментария','комментариев');

echo plural_form(0, $c) . '<br />';
echo plural_form(1, $c) . '<br />';
echo plural_form(2, $c) . '<br />';
echo plural_form(5, $c) . '<br />';
echo plural_form(7, $c) . '<br />';
echo plural_form(25, $c) . '<br />';
```
