---
layout: post
title: Drupal programmaticaly sync CCK fileds between nodes

tags: [action, cck, d6, db_fetch_object, db_query, db_result, drupal, hook_form_alter, hook_rules_action_info, node_save, rules]
---

My case.

When user registers via specific path - he gets seller role (auto assign role module), `about_company` node creates (rules module).

Seller can edit own `about_company` node, but not create or delete.

It works like content profile module, but i do not use him - because i have many other roles, that have other profiles.

About company node has price min, avg, max fields that must be updated when user create, update or delete product nodes that have price field.

**sync_company_prices.info**

```ini
name = Sync company prices
description = Sync company prices, when author make something with products
package = Custom
core = 6.x
```

**sync_company_prices.module**

```php
<?php
// registering our custom action (will be shown when configuring rules)
function sync_company_prices_rules_action_info() {
    return array(
    'sync_company_prices_action' => array(
        'label' => t('Sync company prices'),
        'arguments' => array( // action will recieve product node and its author
        'node' => array('type' => 'node', 'label' => t('Content')),
        'author' => array('type' => 'user', 'label' => t('User, which is set as author')),
        ),
        'module' => 'Node',
    ),
    );
}

// custom action
function sync_company_prices_action($node, $author) {
    // step 1. retrieve node id of company node
    $nid = db_result(db_query("SELECT nid FROM {node} WHERE type = 'company' AND uid = %d LIMIT 1", $author->uid));

    // if there is such node - we will update its prices
    if($nid) {
        // fetch price min, avg, max from company products
        // selecting prices from published products by user id
        $prices = db_fetch_object(db_query("
        SELECT
            MAX({content_type_product}.field_price_value) AS price_max,
            AVG({content_type_product}.field_price_value) AS price_avg,
            MIN({content_type_product}.field_price_value) AS price_min
        FROM {node}
        LEFT JOIN {content_type_product} ON
            {node}.nid = {content_type_product}.nid AND
            {node}.vid = {content_type_product}.vid
        WHERE
            {node}.type = 'product' AND
            {node}.status = 1 AND
            {node}.uid = %d
        ", $author->uid));

        // load node, change its prices and save it
        $company_node = node_load($nid);
        $company_node->field_price_min[0]['value'] = $prices->price_min;
        $company_node->field_price_avg[0]['value'] = $prices->price_avg;
        $company_node->field_price_max[0]['value'] = $prices->price_max;
        node_save($company_node);
    }

    return array('node' => $node);
}

// altering company node form - hiding prices fields
function sync_company_prices_form_alter(&$form, $form_state, $form_id) {
    if($form_id == 'company_node_form') {
        // hide price fields, they will be calculated by rules
        $form['field_price_min']['#type'] = 'value';
        $form['field_price_avg']['#type'] = 'value';
        $form['field_price_max']['#type'] = 'value';
    }
}
?>
```

Now u can create rules, like: When user creates new published product call our custom action.

![screenshot](/images/wp/112.png)

![screenshot](/images/wp/24.png)
