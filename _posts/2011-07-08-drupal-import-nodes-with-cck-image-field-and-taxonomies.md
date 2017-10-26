---
layout: post
title: Drupal import nodes with cck image field and taxonomies

tags: [drupal, import, node_save]
---

```php
<?php

chdir('/hosting/mebelfan.com/');
require_once './includes/bootstrap.inc';
drupal_bootstrap(DRUPAL_BOOTSTRAP_FULL);

// Images are stored in temporary folder:
// /hosting/mebelfan.com/sites/default/panstar
// that may be deleted after import

// $item [title, image_filename, manufacturer_term_id, country_term_id, field_width|depth|height]
$items = array();
$items[] = array('Диван и кресло Мальта', 'mjagkaja_mebel_kyrier.com.ua_malta_maltadiv.jpg', 18, 28, 0, 0, 0);
$items[] = array('Аккорд угловой диван', 'mjagkaja_mebel_mebus_acord_ug_26_1.jpg', 18, 28, 0, 0, 0);
$items[] = array('Диван и кресло Саванна', 'mjagkaja_mebel_kyrier.com.ua_savanna_savdiv.jpg', 18, 28, 0, 0, 0);

foreach($items as $item) {
    $node = new StdClass();
    $node->type = 'product';
    $node->uid = 16;
    $node->title = $item[0];
    $node->promote = 1;

    //$node->created = time();
    //$node->changed = $node->created;
    //$node->status = 1; // Published by default
    //$node->promote = 1;
    //$node->sticky = 0;
    //$node->format = 1;       // Filtered HTML

    // add CCK field data
    $node->field_width[0]['value'] = $item[4] == 0 ? '' : $item[4];
    $node->field_depth[0]['value'] = $item[5] == 0 ? '' : $item[5];
    $node->field_height[0]['value'] = $item[6] == 0 ? '' : $item[6];

    /////// FIELD_PHOTO ///////////////////////////////////////
    // Image file path.
    $image = file_directory_path() . '/panstar/' . $item[1];
    // Load up the CCK field. First parameter image field name and second parameter node type. It can be chnaged to any field name and also and node type.
    $field = content_fields('field_photo', 'product');
    // Load up the appropriate validators
    $validators = array_merge(filefield_widget_upload_validators($field), imagefield_widget_upload_validators($field));
    // Store file path.
    $files_path = filefield_widget_file_path($field);
    // Create the file object, replace existing file with new file as source and dest are the same
    $file = field_file_save_file($image, $validators, $files_path, FILE_EXISTS_REPLACE);
    // put the file into node image field.
    $node->field_photo = array( 0 => $file);

    /////// TAXONOMIES ///////////////////////////////////////
    $node->taxonomy[1] = $item[2]; // proizvoditel
    $node->taxonomy[2] = 26; // stil
    $node->taxonomy[3] = $item[3]; // strana
    $node->taxonomy[4] = 163; // tip
    $node->taxonomy[5] = 164; // jkategoria

    // save node
    node_save($node);

    echo '<a href="http://mebelfan.com/node/' . $node->nid . '/edit">http://mebelfan.com/node/' . $node->nid . '/edit</a><br />';
}
```
