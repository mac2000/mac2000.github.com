---
layout: post
title: Drupal alter node form - move taxonomy fields

tags: [drupal, form, hook_form_alter, taxonomy]
---

**product_form_move_fields.info**

```ini
version = "6.x-1.0"
name = "Product form - move fields"
description = "Moves fields on product add/edit page."
core = "6.x"
package = custom
```

**product_form_move_fields.module**

```php
<?php
function product_form_move_fields_form_alter(&$form, $form_state, $form_id) {
    if($form_id == 'product_node_form') {
        // dump field names and ther weights
        _product_form_move_fields_dump_field_weights($form);

        // move taxonomy fields outside fieldset to other fields
        _product_form_move_fields_move_taxonomy_field($form, 1, 2);
        _product_form_move_fields_move_taxonomy_field($form, 2, 2);
        _product_form_move_fields_move_taxonomy_field($form, 3, 2);
        _product_form_move_fields_move_taxonomy_field($form, 4, -3);
        _product_form_move_fields_move_taxonomy_field($form, 5, -3);

        // move other fields
        $form['field_width']['#weight']   = 7;
        $form['field_height']['#weight']  = 8;
        $form['field_depth']['#weight']   = 9;
        $form['field_gallery']['#weight'] = 10;

        // add submit handler
        $form['#submit'][] = 'product_form_move_fields_submit';

        // hide taxonomy block
        $form['taxonomy']['#prefix'] = '<div style="display:none">';
        $form['taxonomy']['#suffix'] = '</div>';
    }
}

function product_form_move_fields_submit($form, &$form_state) {
    _product_form_move_fields_reset_taxonomy_field($form, 1);
    _product_form_move_fields_reset_taxonomy_field($form, 2);
    _product_form_move_fields_reset_taxonomy_field($form, 3);
    _product_form_move_fields_reset_taxonomy_field($form, 4);
    _product_form_move_fields_reset_taxonomy_field($form, 5);
}

function _product_form_move_fields_dump_field_weights(&$form) {
    $m = '<table border="1"><tr><th>weight</th><th>name</th><th>key</th></tr>';
    foreach($form as $k => $v) {
        $element = $v;
        if(!is_array($element)) continue;

        $title = strip_tags(trim((string)$element['#title']));
        if(empty($title)) continue;
        $weight = $element['#weight'] ? $element['#weight'] : 0;

        $m = $m . '<tr><td>' . $weight . '</td><td>' . $title.'</td><td>' . $k . '</td></tr>';
    }
    $m = $m . '</table>';
    drupal_set_message($m);
}

function _product_form_move_fields_move_taxonomy_field(&$form, $vid, $weight = 0) {
    $form['tax' . $vid] = $form['taxonomy'][$vid];
    $form['tax' . $vid]['#weight'] = $weight;
    unset($form['taxonomy'][$vid]);
}

function _product_form_move_fields_reset_taxonomy_field(&$form, $vid) {
    $form_state['values']['taxonomy'][$vid] = $form_state['values']['tax' . $vid];
    unset($form_state['tax' . $vid]);
}
```
