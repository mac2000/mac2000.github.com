---
layout: post
title: WordPress cross site post

tags: [api, cross, ixr, metaweblog, rpc, wordpress, xmlrpc]
---

Programmaticaly create post from one site on another, with custom fields.

```php
<?php
include('./wp-load.php');
include_once(ABSPATH . WPINC . '/class-IXR.php');

$client = new IXR_Client('http://cross2.wp.local.com/xmlrpc.php');

$post = array(
    'post_type' => 'post',
    'title' => 'Automaticaly created post from cross 1',
    'description' => 'Automaticaly created post from cross 1 content goes here',
    'categories' => array('Task'),
    'custom_fields' => array(),
);

$post['custom_fields'][] = array('key' => 'Price', 'value' => '22');
$post['custom_fields'][] = array('key' => 'Good', 'value' => 'YES');

if (!$client->query('metaWeblog.newPost', 0, 'admin', 'PASSWORD', $post, TRUE)) {
    //die('Something went wrong - '.$client->getErrorCode().' : '.$client->getErrorMessage());
    //TODO: redirect here - something wrong
} else {
    $response = $client->getResponse();
    //TODO: all ok
}
```
