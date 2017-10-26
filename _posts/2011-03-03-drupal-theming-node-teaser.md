---
layout: post
title: Drupal theming node teaser

tags: [drupal, node, teaser, template, theme, tpl]
---

node-story.tpl.php

node-page.tpl.php

```php
<?php
if ($teaser) {
    // node is being displayed as a teaser
    // Anything here will show up when the teaser of the post is viewed in your taxonomies or front page
} else {
    //all other cases
    //Anything here will show up when viewing your post at any other time, e.g. previews
}
```

template.php

```php
function phptemplate_preprocess_node(&$vars) {
    $vars['template_files'][] = 'node-' . $vars['nid'];
}
```