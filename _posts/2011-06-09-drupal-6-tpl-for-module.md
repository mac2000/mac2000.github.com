---
layout: post
title: drupal 6 tpl for module

tags: [drupal, hook_theme, module, template, theme, tpl]
---

**cust.info**

```ini
name = cust
description = cust
package = Other
core = 6.x
```

**cust.module**

```php
<?php
function cust_theme() {
    return array(
            'moe_example' => array(
                    'arguments' => array(
                        'some_data' => NULL,
                        ),
                    'template' => 'moe-example',
            ),
    );
}
?>
```

**moe-example.tpl.php**

```php
start<br />
<?php var_dump($some_data);?>
end<br />
```

somewhere in **node.tpl.php**

```php
<?php echo 'xx';
print theme('moe_example', array(1,2,3));
?>
```

http://www.captaincodemonkey.com/blog/2010/12/26/drupal-6x-skeleton-module-with-tpl-file-example/

And more complex example:

**catalog.info**

```ini
name = catalog
description = "custom catalog module."
core = 6.x
```

**catalog.module**

```php
<?php
//pull in our include file
module_load_include('inc', 'catalog');

function catalog_perm() {
    return array('add items');
}

//lets get some sweet sweet menu action going
function catalog_menu() {

    $items['catalog'] = array(
        'title' => 'My catalog example',
        'description' => t('My catalog example desc'),
        'page callback' => 'catalog_page', //
        'access callback' => 'user_access',
        'access arguments' => array('access content'), // or use hook_perm() to make your own
        'type' => MENU_CALLBACK,
    );
    return $items;
}

//function
function catalog_page() {
    $output = theme('catalog_catalog', _catalog_get_items());

    if(user_access('add item')) {
        $output = $output . theme('catalog_form');
    }
    return $output;
}

//hook_theme
function catalog_theme($existing, $type, $theme, $path) {
    return array(
        'catalog_catalog' => array(
            'arguments' => array('items' => NULL),
            'template' => 'catalog-catalog',
        ),
        'catalog_form' => array(
            'template' => 'catalog-form',
        ),
    );
}
```

**catalog.inc**

```php
<?php

/* Put any helper functions or other stuff you want in here */
function _catalog_get_items() {
    return array(
        array(
            'id' => 1,
            'name' => 'one'
        ),
        array(
            'id' => 2,
            'name' => 'two'
        )
    );
}

?>
```

**catalog-catalog.tpl.php**

```
hello from catalog.tpl.php
```

**catalog-form.tpl.php**

```
hello from catalog-form.tpl.php
```

Now if you will go to example.com/catalog - you will see both templates if you logged as admin, or one if not logged or have not privilegies to add items.
