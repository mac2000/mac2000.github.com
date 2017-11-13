---
layout: post
title: WooCommerce Categories Menu with Images
tags: [wordpress, woocommerce, widget, WC_Product_Cat_List_Walker]
---

Adding categories with images menu widget to head of site

<amp-img src="/images/woo-categories-menu-with-images.png" alt="screenshot" width="1062" height="380"></amp-img>

(Do not take it hard from visual point of view, it is WIP)

To make such things you need:

Register widget
---------------

It is almost straigh forward copy paste from `WC_Widget_Product_Categories` unfortunatelly there is no way to extend it

```php
<?php

class Trofi_Widget_Product_Categories_Menu extends WC_Widget
{

    public $cat_ancestors;

    public $current_cat;

    public function __construct()
    {
        $this->widget_cssclass = 'trofi cat-menu';
        $this->widget_description = __('A list of product categories.', 'trofi');
        $this->widget_id = 'trofi_product_categories';
        $this->widget_name = __('Product Categories Menu', 'trofi');
        $this->settings = array(
            'title' => array(
                'type' => 'text',
                'std' => __('Product Categories', 'woocommerce'),
                'label' => __('Title', 'woocommerce')
            ),
            'orderby' => array(
                'type' => 'select',
                'std' => 'name',
                'label' => __('Order by', 'woocommerce'),
                'options' => array(
                    'order' => __('Category Order', 'woocommerce'),
                    'name' => __('Name', 'woocommerce')
                )
            )
        );

        parent::__construct();
    }

    public function widget($args, $instance)
    {
        global $wp_query, $post;

        $orderBy = isset($instance['orderby']) ? $instance['orderby'] : $this->settings['orderby']['std'];

        require_once(trailingslashit(get_stylesheet_directory()) . 'walkers/Trofi_Product_Cat_Menu_Walker.php');

        $list_args = [
            'show_count' => false,
            'hierarchical' => true,
            'taxonomy' => 'product_cat',
            'hide_empty' => true,
            'menu_order' => false,
            'title_li' => '',
            'pad_counts' => 1,
            'show_option_none' => __('No product categories exist.', 'woocommerce'),
            'walker' => new Trofi_Product_Cat_Menu_Walker
        ];

        // Menu Order
        if ($orderBy == 'order') {
            $list_args['menu_order'] = 'asc';
        } else {
            $list_args['orderby'] = 'title';
        }

        // Setup Current Category
        $this->current_cat = false;
        $this->cat_ancestors = array();

        if (is_tax('product_cat')) {
            $this->current_cat = $wp_query->queried_object;
            $this->cat_ancestors = get_ancestors($this->current_cat->term_id, 'product_cat');
        } elseif (is_singular('product')) {
            $product_category = wc_get_product_terms($post->ID, 'product_cat', array('orderby' => 'parent'));

            if ($product_category) {
                $this->current_cat = end($product_category);
                $this->cat_ancestors = get_ancestors($this->current_cat->term_id, 'product_cat');
            }
        }
        $list_args['current_category'] = ($this->current_cat) ? $this->current_cat->term_id : '';
        $list_args['current_category_ancestors'] = $this->cat_ancestors;


        $this->widget_start($args, $instance);
        echo '<ul class="product-categories">';
        wp_list_categories(apply_filters('woocommerce_product_categories_widget_args', $list_args));
        echo '</ul>';
        $this->widget_end($args);
    }
}
```

in your `functions.php`:

```php
add_action('widgets_init', function () {
    require_once(trailingslashit(get_stylesheet_directory()) . 'widgets/Trofi_Widget_Product_Categories_Menu.php');
    register_widget('Trofi_Widget_Product_Categories_Menu');
});
```

The key difference here is that we use our custom walker `Trofi_Product_Cat_Menu_Walker` which will render category image for top level categories.

Walker
------

Here we are extending WooCommerce product categories list walker and overriding its `start_el` method. The only difference from parent method if that we are adding category images inside anchors for top level categories.

```php
<?php
if (!defined('ABSPATH')) {
    exit; // Exit if accessed directly
}

require_once(ABSPATH . '/wp-content/plugins/woocommerce/includes/walkers/class-product-cat-list-walker.php');

class Trofi_Product_Cat_Menu_Walker extends WC_Product_Cat_List_Walker
{
    public function start_el(&$output, $cat, $depth = 0, $args = array(), $current_object_id = 0)
    {
        $output .= '<li class="cat-item cat-item-' . $cat->term_id;

        if ($args['current_category'] == $cat->term_id) {
            $output .= ' current-cat';
        }

        if ($args['has_children'] && $args['hierarchical']) {
            $output .= ' cat-parent';
        }

        if ($args['current_category_ancestors'] && $args['current_category'] && in_array($cat->term_id, $args['current_category_ancestors'])) {
            $output .= ' current-cat-parent';
        }

        //$output .=  '"><a href="' . get_term_link( (int) $cat->term_id, 'product_cat' ) . '">' . __($cat->name, 'woocommerce') . '</a>';
        $output .= '"><a href="' . get_term_link((int)$cat->term_id, 'product_cat') . '">';
        if ($depth === 0) {
            $category_thumbnail = get_woocommerce_term_meta($cat->term_id, 'thumbnail_id', true);
            $image = wp_get_attachment_url($category_thumbnail);
            $image = $image ? $image : get_theme_mod('trofi_category_default_image');
            $image = $image ? $image : 'http://placehold.it/100x100&text=No+photo';
            $output .= '<span class="category-image" style="background-image:url(' . esc_attr($image) . ')"></span>';
        }
        $output .= __($cat->name, 'woocommerce') . '</a>';
    }
}
```


And thats all, now you can drop widget to header area or call it from `functions.php`

```php
add_action('init', function () {
    add_action('storefront_header', function () {
        the_widget('Trofi_Widget_Product_Categories_Menu');
    }, 60);
});
```

All is left is to have fun with styling it :)
