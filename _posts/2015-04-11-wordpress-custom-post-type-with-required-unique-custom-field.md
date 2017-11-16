---
layout: post
title: Wordpress Custom Post Type with Required Unique Custom Field
tags: [wordpress, add_meta_box, wp_insert_post_data, redirect_post_location, admin_notices]
---

We are going to add custom video post type to wordpress with required and unique video url custom field.

So user will be able to add new video posts only if he enters video url and it is unique (meaning that there is no other posts with same video url) otherwise we will notify user about stufs that are went wrong and save post as draft so user can fix things and publish post.

Short note: There is nothing special in code, and even more it is not good and should not be used in production as is it is just proof of concept.

So all code is placed in `wp-content/plugins/custom_post_type_with_required_custom_field.php`

With pretty standard header:

```
/*
Plugin Name: Custom Post Type With Required Custom Field
Plugin URI: http://mac-blog.org.ua/
Description: Sample plugin demonstrating how to add required unique custom field to custom post type
Author: Marchenko Alexandr
Version: 1.0
Author URI: http://mac-blog.org.ua/
*/
```

We have some constants that we are using later in code, just for not being messing up with strings

```php
define('CPTWRCF_POST_TYPE', 'video');
define('CPTWRCF_META_KEY', '_video');
define('CPTWRCF_META_LABEL', 'Video URL');

define('CPTWRCF_NONCE_ACTION', 'cptwrcf_nonce_action');
define('CPTWRCF_NONCE_NAME', 'cptwrcfn_nonce_name');

define('CPTWRCF_NOT_VALID_QUERY_STRING_KEY', 'cptwrcfn_not_valid');
define('CPTWRCF_NOT_UNIQUE_QUERY_STRING_KEY', 'cptwrcfn_not_unique');

define('CPTWRCF_SQL', "SELECT * FROM $wpdb->postmeta WHERE meta_key = %s AND meta_value = %s AND post_id <> %d");
```

First things first we are going to create custom post type

```php
add_action('init', function() {
    register_post_type(CPTWRCF_POST_TYPE, [
        'label' => 'Video',
        'public' => true,
        'supports' => ['title', 'editor', 'thumbnail', 'excerpt', 'comments'],
        'show_in_menu' => 'edit.php',
        'menu_icon' => 'dashicons-video-alt3',
        //'taxonomies' => [''],
        'has_archive' => true,
    ]);
});
```

And add custom meta box for its Video URL custom field

```php
add_action('add_meta_boxes', function() {
    add_meta_box(CPTWRCF_META_KEY, CPTWRCF_META_LABEL, function(WP_Post $post) {
        wp_nonce_field(CPTWRCF_NONCE_ACTION, CPTWRCF_NONCE_NAME);

        $attributes = [
            'id' => CPTWRCF_META_KEY,
            'name' => CPTWRCF_META_KEY,
            'value' => get_post_meta($post->ID, CPTWRCF_META_KEY, true),
            'class' => 'widefat',
            'type' => 'text',
            //'type' => 'url',
            //'required' => 'required'
        ];

        $attributes = implode(' ', array_map(function($key) use($attributes) {
            return sprintf('%s="%s"', $key, esc_attr($attributes[$key]));
        }, array_keys($attributes)));

        echo '<input ' . $attributes . ' />';
    }, CPTWRCF_POST_TYPE, 'side', 'core');
});
```

Handle post save event and save Video URL custom field as post meta

```php
add_action('save_post', function($post_id) {
    if (!isset($_POST[CPTWRCF_NONCE_NAME])) return;
    if (!wp_verify_nonce($_POST[CPTWRCF_NONCE_NAME], CPTWRCF_NONCE_ACTION)) return;
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) return;
    if (!isset($_POST[CPTWRCF_META_KEY])) return;
    if (!current_user_can('edit_post', $post_id)) return;

    update_post_meta($post_id, CPTWRCF_META_KEY, sanitize_text_field($_POST[CPTWRCF_META_KEY]));
});
```

All that was pretty standard stuff with nothing special

Now we are going to add two additional capabilities: Make Video URL custom field - required and ensure that it is unique across all posts

Required Custom Field
---------------------

<amp-img src="/images/cptwrcf/required.png" alt="Required custom field notice" width="870" height="654"></amp-img>

```php
add_filter('wp_insert_post_data', function (array $data) {
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) return $data;
    if(!empty($_GET['action']) && $_GET['action'] === 'trash') return $data; // Make sure to do nothing for posts that are going to be deleted

    // If there is not Video URL or it is not valid URL - mark post as draft and notify user
    if(empty($_POST[CPTWRCF_META_KEY]) || !filter_var($_POST[CPTWRCF_META_KEY], FILTER_VALIDATE_URL)) {
        $data['post_status'] = 'draft';
        add_filter('redirect_post_location', function($location) {
            $location = remove_query_arg('message', $location);
            $location = add_query_arg('message', 10, $location); // 10 is for "Post draft updated" message
            return add_query_arg(CPTWRCF_NOT_VALID_QUERY_STRING_KEY, 1, $location);
        });
    } else {
        add_filter('redirect_post_location', function($location) {
            return remove_query_arg(CPTWRCF_NOT_VALID_QUERY_STRING_KEY, $location);
        });
    }

    return $data;
});

add_action('admin_notices', function () {
    if (isset($_GET[CPTWRCF_NOT_VALID_QUERY_STRING_KEY])) {
        $link = sprintf('<b><a href="#%s">%s</a></b>', CPTWRCF_META_KEY, CPTWRCF_META_LABEL);
        $message = sprintf(__('Your post was saved as draft because there is no required %s or it is invalid!'), $link);
        echo sprintf('<div class="error"><p>%s</p></div>', $message);
    }
});
```

Unique Custom Field
-------------------

<amp-img src="/images/cptwrcf/unique.png" alt="Unique custom field notice" width="870" height="654"></amp-img>

```php
add_filter('wp_insert_post_data', function (array $data, array $raw) {
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) return $data;
    if(!empty($_GET['action']) && $_GET['action'] === 'trash') return $data; // Make sure to do nothing for posts that are going to be deleted

    $post_id = (empty($data['ID']) ? $raw['ID'] : $data['ID']) ?: 0;

    /** @var wpdb $wpdb */
    global $wpdb;
    $query = $wpdb->prepare(CPTWRCF_SQL, CPTWRCF_META_KEY, @$_POST[CPTWRCF_META_KEY], $post_id);
    $found = $wpdb->get_results($query, ARRAY_A);

    // If we found posts with save Video URL - notify user and save post as a draft
    if($found) {
        $data['post_status'] = 'draft';
        add_filter('redirect_post_location', function ($location){
            $location = remove_query_arg('message', $location);
            $location = add_query_arg('message', 10, $location); // 10 is for "Post draft updated" message from `edit-form-advanced.php`
            return add_query_arg(CPTWRCF_NOT_UNIQUE_QUERY_STRING_KEY, 1, $location);
        });
    } else {
        add_filter('redirect_post_location', function ($location){
            return remove_query_arg(CPTWRCF_NOT_UNIQUE_QUERY_STRING_KEY, $location);
        });
    }

    return $data;
}, 10, 2); // Notice last arg: 2 - is for getting raw data which is used to determine post id

add_action('admin_notices', function () {
    if (isset($_GET[CPTWRCF_NOT_UNIQUE_QUERY_STRING_KEY])) {
        /** @var wpdb $wpdb */
        global $wpdb;

        /** @var WP_Post $post */
        global $post;

        // <editor-fold desc="Optional: Get post links that has same Video URL">
        $meta_value = get_post_meta($post->ID, CPTWRCF_META_KEY, true);
        $query = $wpdb->prepare(CPTWRCF_SQL, CPTWRCF_META_KEY, $meta_value, $post->ID);
        $items = $wpdb->get_results($query, ARRAY_A);
        $items = array_map(function ($item) {
            return sprintf('<a target="_blank" href="%s">%s</a>', get_permalink($item['post_id']), get_the_title($item['post_id']));
        }, $items);
        $items = implode(', ', $items);
        $used_by = sprintf('by %s', $items);
        // </editor-fold>

        $link = sprintf('<b><a href="#%s">%s</a></b>', CPTWRCF_META_KEY, CPTWRCF_META_LABEL);
        $message = sprintf(__('Your post was saved as draft because %s is already used %s'), $link, $used_by);
        echo sprintf('<div class="error"><p>%s</p></div>', $message);
    }
});
```

The first thing you will ask - why the heck `wp_insert_post_data` and `admin_notices` and you will be totally right :)

If you need required **and** unique custom field you should combine them, but if you need only required field just use first part, as I sad it is just proof of concept and not production code (probably it all should be rewriten in some cleaner OOP way but seems to be too heavy for small note)

Full code can be found here: https://gist.github.com/mac2000/68721ed47567c469fc83
