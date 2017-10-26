---
layout: post
title: Drupal taxonomy name in $body_classes

tags: [bodyclasses, class, css, drupal, php, style, taxonomy, template, theme, tpl]
---

Edit `template.php`, `phptemplate_preprocess_page` function and add:

```php
$path = drupal_get_path_alias($_GET['q']);
if(strpos($path, 'category/') === 0) {
    $path = explode('/', $path);
    if(isset($path[1]) && !empty($path[1])) {
        $taxonomy_name = $path[1];
        $taxonomy_name = strtolower(preg_replace('/[^a-zA-Z0-9_-]+/', '-', $taxonomy_name));
        $taxonomy_name = 'taxonomy-' . $taxonomy_name;
        $vars['body_classes'] .= ' ' . $taxonomy_name;
    }
}
```
