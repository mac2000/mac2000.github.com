---
layout: post
title: Drupal computed_field

tags: [cck, computed, field, drupal]
---

Note about using `compute_field`.

**Modules:**

http://drupal.org/project/cck - with multigroup (at moment in dev)

http://drupal.org/project/computed_field

**Node types:**

**Product**

Product have one additional field `Price` that is required and can be only one per product.

![screenshot](/images/wp/118.png)

![screenshot](/images/wp/119.png)

![screenshot](/images/wp/29.png)

Order

Order have noderefference Product field and its amount in Multigroup and computed field Total price.

![screenshot](/images/wp/34.png)

![screenshot](/images/wp/43.png)

![screenshot](/images/wp/53.png)

![screenshot](/images/wp/62.png)

**Computed field snippet:**

```php
$total_price = 0;
foreach($node->field_order_product as $k => $v) {
  $product = node_load($node->field_order_product[$k]['nid']);
  $product_price = $product->field_product_price[0]['value'];
  $amount = $node->field_order_product_amount[$k]['value'];

  $total_price += $product_price * $amount;
}

$node_field[0]['value'] = $total_price;
```

More snippet examples:

[http://drupal.org/node/149228](http://drupal.org/node/149228)

More complex example, added field _Discount_ and computed field _Price_ to _Multigroup_, computed field _Price_ will be calculated foreach selected product. Computed field _Total price_ also changed to retrive price with discount.

![screenshot](/images/wp/82.png)

![screenshot](/images/wp/92.png)

![screenshot](/images/wp/93.png)

**Code snippet for Price:**

```php
foreach (array_keys($node->field_order_product) as $delta) {
  $product = node_load($node->field_order_product[$delta]['nid']);
  $product_price = $product->field_product_price[0]['value'];
  $amount = $node->field_order_product_amount[$delta]['value'];
  $discount = $node->field_discount[$delta]['value'];
  $node_field[$delta]['value'] = ($product_price * $amount) * ((100 - $discount) / 100);
}
```

**Code snippet for Total Price:**

```php
$total_price = 0;
foreach($node->field_order_product_price as $order_product_price) {
  $total_price += $order_product_price['value'];
}

$node_field[0]['value'] = $total_price;
```
