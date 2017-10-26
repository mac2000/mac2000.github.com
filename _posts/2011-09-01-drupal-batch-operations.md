---
layout: post
title: Drupal batch operations

tags: [batch, batch_process, batch_set, d6, drupal, hook_menu, job]
---

Example of script for internal use

**macbatch.info**

```ini
name = "macbatch"
core = "6.x"
version = "6.x-1.0"
```

**macbatch.module**

```php
<?php
function macbatch_menu() {
    return array(
        'macbatch' => array(
            'title' => 'Batch',
            'page callback' => 'macbatch_page_callback',
            'access arguments' => array('administer access control'),
            'type' => 'MENU_CALLBACK',
        )
    );
}

function _macbatch_finished($success, $results, $operations) {
    if ($success) {
        $message = count($results) .' processed.';
        $message .= theme('item_list', $results);
    }
    else {
        $error_operation = reset($operations);
        $message = t('An error occurred while processing %error_operation with arguments: @arguments', array('%error_operation' => $error_operation[0], '@arguments' => print_r($error_operation[1], TRUE)));
    }
    drupal_set_message($message);
}

function _macbatch_run(array $operations) {
    batch_set(array(
        'operations' => $operations,
        'finished' => '_macbatch_finished'
    ));
    batch_process('node');
}

function macbatch_page_callback() {
    if(function_exists('_' . arg(1))) call_user_func('_' . arg(1));
    else echo 'not found';
}

/////////////////////////////////////////////////////

// will be accessible on url: http://example.com/macbatch/geo
function _geo() {
    // fill $operations arrray
    $operations = array();
    $q = db_query("SELECT nid FROM {node} WHERE type = 'salon'");
    while ($r = db_fetch_object($q)) {
        $operations[] = array('_geo_process', array($r->nid));
    }
    // and run batch on them
    _macbatch_run($operations);
}

// operation
function _geo_process($nid, &$context) {
    $node = node_load($nid);
    //sleep(1);
    $context['results'][] = check_plain($node->title);
}
```

To create some batch job - u must define `YOUR_JOB_NAME` and `YOUR_JOB_NAME_process` functions.

First must fill operations array, which will contains second function name, and array with params.

Second will actualy do job.
