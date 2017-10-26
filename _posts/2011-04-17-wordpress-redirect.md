---
layout: post
title: WordPress redirect

tags: [access, acl, auth, get_post_meta, redirect, template_redirect, user, wordpress, wp_get_post_categories, wp_redirect]
---

In sample code, we have posts of category "task", that have "author" custom field, and redirect not logged users or users that not assigned to this task.

```php
function test_redirect(){
    if(is_single()){
        global $post;
        global $current_user;
        $current_post_categories = wp_get_post_categories($post->ID);

        $is_task = false; // <--

        if (in_array(get_cat_id('task'), $current_post_categories)) {
            $is_task = true;
        }

        $author = get_post_meta($post->ID, 'Author', true); // <--
        $current_user_login = ""; // <--

        if($current_user->data != NULL) {
            $current_user_login = $current_user->data->user_login;
        }

        if($is_task == true && $author != "" && $current_user_login != $author && $current_user_login != "admin") {
            wp_redirect( get_bloginfo('siteurl') . '/notallowed/' );
            exit;
        }
    }
}
add_action('template_redirect', 'test_redirect');
```

Also `template_redirect` can be used to assign custom template files. See: http://www.mihaivalentin.com/wordpress-tutorial-load-the-template-you-want-with-template_redirect/
