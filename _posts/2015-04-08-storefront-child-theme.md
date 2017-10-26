---
layout: post
title: StoreFront Child Theme
tags: [storefront, child, theme, woo, woocommerce, override]
---

**wp-content/themes/storefront-child/styles.css**

```css
/*
    Theme Name:   My
    Theme URI:    http://example.com/twenty-fifteen-child/
    Description:  Extending StoreFront
    Author:       Marchenko Alexandr
    Author URI:   http://mac-blog.org.ua
    Template:     storefront
    Version:      1.0.0
    License:      MIT
*/
```

**wp-content/themes/storefront-child/functions.php**

```php
add_filter('storefront_credit_link', '__return_false');

function theme_enqueue_styles() {
    wp_deregister_style('storefront-style');
    wp_enqueue_style('storefront-style', get_template_directory_uri() . '/style.css' );
    wp_enqueue_style('storefront-child-style', get_stylesheet_uri(), ['storefront-style', 'storefront-woocommerce-style']);
}
add_action('wp_enqueue_scripts', 'theme_enqueue_styles');
```

Unfortunatelly I did not found a way to not to use deregister storefron style, otherwise you will get or only your child theme style or your styles will load twice.

Also notice that our style is depended on `storefront-woocommerce-style` so our style loads after `woocommerce.css` which is pretty handy for overriding woocommerce styles.
