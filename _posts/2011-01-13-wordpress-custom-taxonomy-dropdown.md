---
layout: post
title: WordPress custom taxonomy dropdown

tags: [dropdown, php, taxonomy, template, theme, wordpress]
---

Taxonomies in wp can be used as tags (simple input) or as categories (list of checkboxes).

Here is code, to add custom taxonomy that will allow only one selection.

**functions.php**

```php
function register_my_taxonomies() {
    register_taxonomy('mytaxname', 'post', array(
        'label' => 'My taxonomy',
        'hierarchical' => true,
    ));
}
add_action('init', 'register_my_taxonomies', 0 );
function my_admin_scripts() {
    wp_register_script('my_admin_scripts', get_bloginfo('template_url') . '/my_admin_scripts.js', array('jquery'));
    wp_enqueue_script('my_admin_scripts');
}
add_action('admin_print_scripts', 'my_admin_scripts');
```


**my_admin_scripts.js**

```js
jQuery(document).ready(function($) {
    var opts = '<option value="0">- не указан -</option>';
    $('#taxonomy-priority #prioritychecklist label.selectit').each(function(){
        var v = $(this).find('input:checkbox').val();
        var k = $(this).text().trim();
        var c = $(this).find('input:checkbox').get(0).checked ? ' selected="selected" ' : '';
        opts += '<option value="'+v+'" '+c+'>'+k+'</option>';
    });
    $('<div id="taxonomy-priority-dropdown-holder"><select id="taxonomy-priority-dropdown" style="width:100%;">'+opts+'</select></div>').insertAfter('#taxonomy-priority');

    $('#taxonomy-priority-dropdown').change(function(){
        var val = $(this).val();
        $('#prioritychecklist input:checkbox').each(function(){
            var v = $(this).val();
            var k = $(this).parent().text().trim();
            var c = this.checked;
            if(c == true && c != val) $(this).click();
            if(c == false && v == val) $(this).click();
        });
    });
    $('#taxonomy-priority').hide();
});
```

Some fix for many custom taxonomies:

```js
jQuery(document).ready(function($) {
    make_dropdown($, 'priority');
    make_dropdown($, 'relevance');
});

function make_dropdown($, taxonomy) {
    var opts = '<option value="0">- n/a -</option>';
    $('#taxonomy-'+taxonomy+' #'+taxonomy+'checklist label.selectit').each(function(){
        var v = $(this).find('input:checkbox').val();
        var k = $(this).text().trim();
        var c = $(this).find('input:checkbox').get(0).checked ? ' selected="selected" ' : '';
        opts += '<option value="'+v+'" '+c+'>'+k+'</option>';
    });
    $('<div id="taxonomy-'+taxonomy+'-dropdown-holder"><select id="taxonomy-'+taxonomy+'-dropdown" style="width:100%;">'+opts+'</select></div>').insertAfter('#taxonomy-'+taxonomy);

    $('#taxonomy-'+taxonomy+'-dropdown').change(function(){
        var val = $(this).val();
        $('#'+taxonomy+'checklist input:checkbox').each(function(){
            var v = $(this).val();
            var k = $(this).parent().text().trim();
            var c = this.checked;
            if(c == true && c != val) $(this).click();
            if(c == false && v == val) $(this).click();
        });
    });
    $('#taxonomy-'+taxonomy).hide();
}
```
