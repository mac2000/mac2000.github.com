---
layout: post
title: Modify wordpress admin

tags: [admin, administrator, php, template, theme, wordpress, wp]
---

Hide unused panels, dashbords etc

```php
function remove_dashboard_widgets(){
  global $wp_meta_boxes;
  unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_plugins']);
  unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_recent_comments']);
  unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_primary']);
  unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_incoming_links']);
  unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_right_now']);
  unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_secondary']);
}
add_action('wp_dashboard_setup', 'remove_dashboard_widgets');

function remove_menu_items() {
  global $menu;
  //$restricted = array(__('Links'), __('Comments'), __('Media'), __('Plugins'), __('Tools'), __('Users'));
  $restricted = array(__('Links'), __('Comments'), __('Media'), __('Tools'), __('Users'));
  end ($menu);
  while (prev($menu)) {
    $value = explode(' ',$menu[key($menu)][0]);
    if(in_array($value[0] != NULL?$value[0]:"" , $restricted)){
      unset($menu[key($menu)]);}
  }
}
add_action('admin_menu', 'remove_menu_items');

function customize_meta_boxes() {
  /* Removes meta boxes from Posts */
  remove_meta_box('postcustom','post','normal');
  remove_meta_box('trackbacksdiv','post','normal');
  remove_meta_box('commentstatusdiv','post','normal');
  remove_meta_box('commentsdiv','post','normal');
  remove_meta_box('authordiv','post','normal');
  remove_meta_box('revisionsdiv','post','normal');
  //remove_meta_box('tagsdiv-post_tag','post','normal');
  remove_meta_box('postexcerpt','post','normal');
  /* Removes meta boxes from pages */
  remove_meta_box('postcustom','page','normal');
  remove_meta_box('trackbacksdiv','page','normal');
  remove_meta_box('commentstatusdiv','page','normal');
  remove_meta_box('commentsdiv','page','normal');
  remove_meta_box('authordiv','page','normal');
  remove_meta_box('revisionsdiv','page','normal');

}
add_action('admin_init','customize_meta_boxes');

function custom_post_columns($defaults) {
  unset($defaults['comments']);
  unset($defaults['author']);
  return $defaults;
}
add_filter('manage_posts_columns', 'custom_post_columns');

function custom_pages_columns($defaults) {
  unset($defaults['comments']);
  unset($defaults['author']);
  return $defaults;
}
add_filter('manage_pages_columns', 'custom_pages_columns');
```