---
layout: post
title: WP custom taxonomies for post

tags: [category, post, register_taxonomy, taxonomy, wordpress, wp]
---

![screenshot](/images/wp/19.png)

in functions.php:

```php
function register_custom_taxonomies(){
    register_taxonomy( 'notetype', 'post', array('label' => 'Тип заметки', 'hierarchical' => true));
    register_taxonomy( 'customer', 'post', array('label' => 'Заказчик', 'hierarchical' => true));
}
add_action( 'init', 'register_custom_taxonomies', 0 );
```

in single.php:

```php
<ul>
    <?php $categories = wp_get_post_terms(get_the_ID(), 'notetype');foreach($categories as $c) {?>
        <li><a href="<?php echo get_term_link($c, $c->taxonomy)?>"><?php echo $c->name?> (<?php echo $c->count?>)</a></li>
    <?php } ?>
</ul>

<ul>
    <?php $categories = wp_get_post_terms(get_the_ID(), 'customer');foreach($categories as $c) {?>
        <li><a href="<?php echo get_term_link($c, $c->taxonomy)?>"><?php echo $c->name?> (<?php echo $c->count?>)</a></li>
    <?php } ?>
</ul>
```
