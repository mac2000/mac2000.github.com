---
layout: post
title: Drupal 6 programatically change location of comment submissiom form
tags: [drupal, comment, preprocess_node, comment_form_location, comment_form_below, comment_form_separate_page]
---

Here is example how to change location of comment submissiom form depending on node sticky flag (can be anything you want)

Add to your `template.php` file next function:

```php
// THEME_preprocess_node
function test_preprocess_node(&$vars, $hook) {
    variable_set('comment_form_location_' . $vars['node']->type, $vars['node']->sticky ? COMMENT_FORM_SEPARATE_PAGE : COMMENT_FORM_BELOW );
}
```

Here is more complex example:

```php
// THEME_preprocess_node
function test_preprocess_node(&$vars, $hook) {
    // Show\hide comment form depending on node sticky flag
    variable_set('comment_form_location_' . $vars['node']->type, $vars['node']->sticky ? COMMENT_FORM_SEPARATE_PAGE : COMMENT_FORM_BELOW );
    // Render comments to $comments variable wich will be available in your node template file
    $vars['comments'] = comment_render($vars['node']);
    // Do not render comments in page template
    $vars['node']->comment = 0;
}
```

Starting point was found here: http://adaptivethemes.com/print-comments-anywhere-and-the-form-as-well
