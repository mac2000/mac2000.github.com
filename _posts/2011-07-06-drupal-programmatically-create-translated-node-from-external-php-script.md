---
layout: post
title: Drupal programmatically create translated node from external php script

tags: [bootstrap, d6, drupal, external, i18n, l10n, multilanguage, node_save, translate]
---

```php
<?php

//set the working directory to your Drupal root
chdir('/home/mac/Projects/drupal/multi/');

//require the bootstrap include
require_once './includes/bootstrap.inc';

//Load Drupal
drupal_bootstrap(DRUPAL_BOOTSTRAP_FULL);
//(loads everything, but doesn't render anything)

// create node in default language
$node = new StdClass();
$node->type = 'product';
$node->uid = 1;
$node->title = 'Product 4 title ru';
$node->body = 'Product 4 body ru';

$node->promote = 1;
/*$node->created = time();
$node->changed = $node->created;
$node->status = 1; // Published by default
$node->promote = 1;
$node->sticky = 0;
$node->format = 1;       // Filtered HTML
*/
$node->language = 'ru';

// add CCK field data
$node->field_code[0]['value'] = 'code for product 4 ru';

// save node
node_save($node);

// modify node:tnid - used by i18n
$tnid = $node->nid;
$node->tnid = $tnid;
node_save($node);

// create translation
$node = new StdClass();
$node->type = 'product';
$node->uid = 1;
$node->title = 'Product 4 title en';
$node->body = 'Product 4 body en';
$node->promote = 1;
$node->language = 'en';
$node->field_code[0]['value'] = 'code for product 4 en';
$node->tnid = $tnid;
node_save($node);

//display a node
print '<pre>';
print_r($node);
print '</pre>';
```
