---
layout: post
title: WordPress Settings API, text, tinymce, file upload simple example

tags: [add_action, add_option, add_settings_section, add_settings_field, add_theme_page, admin_init, admin_menu, do_settings_sections, get_option, options, register_settings, settings, settings_fields, submit_button, wordpress]
---

Here is example screenshot:

![screenshot](/images/wp/example.png)

```php
<?php
// return array of default theme options
function example_get_default_theme_options() {
    $options = array(
        'slogan' => 'Hello world!',
        'contacts' => 'Provide your contact information here',
        'copyright' => 'All rights reserverd.',
        'logotype' => '',
    );
    return $options;
}

// simple helper functions to get option values in theme
function get_example_option($name) {
    $options = get_option('example_theme_options', example_get_default_theme_options());

    return $options[$name];
}
function example_option($name) {
    echo get_example_option($name);
}

// add theme settings page
function example_menu_options() {
    // page title, menu title, access rules, url slug, render callback function
    $page = add_theme_page('Example Options', 'Example', 'edit_theme_options', 'theme_options', 'example_theme_options_render_page');
    add_action('admin_print_scripts-' . $page, 'example_options_page_enqueue_assets');
}
add_action('admin_menu', 'example_menu_options');

// enqueue needed assets (for tinymce actualy)
function example_options_page_enqueue_assets() {
    wp_enqueue_script('common');
    wp_enqueue_script('jquery-color');
    wp_print_scripts('editor');
    if (function_exists('add_thickbox')) add_thickbox();
    wp_print_scripts('media-upload');
    if (function_exists('wp_tiny_mce')) wp_tiny_mce();
    wp_admin_css();
    wp_enqueue_script('utils');
    do_action("admin_print_styles-post-php");
    do_action('admin_print_styles');

    // handling upload_button for logotype, copy-pasted from custom-metadata plugin
    ?>
    <script type="text/javascript">
    jQuery(document).ready(function($) {
        var upload_field, upload_preview;
        if ($('.upload_button').length) {
            $('.upload_button').live('click', function(e) {
                upload_field = $(this).closest('td').find('input.upload_field:first');
                upload_preview = $(this).closest('td').find('img.upload_preview:first');
                window.send_to_editor=window.send_to_editor_clone;
                tb_show('','media-upload.php?TB_iframe=true');
                return false;
            });
            window.original_send_to_editor = window.send_to_editor;
            window.send_to_editor_clone = function(html){
                file_url = jQuery('img',html).attr('src');
                if (!file_url) { file_url = jQuery(html).attr('href'); }
                tb_remove();
                upload_field.val(file_url);
                upload_preview.attr('src', file_url);
            }
        }

        $('.upload_clear').live('click', function(e) {
            $(this).closest('td').find('input.upload_field:first').val('');
            $(this).closest('td').find('img.upload_preview:first').hide();
            return false;
        });
    });
    </script>
    <?php
}

// register settings
function example_settings_api_init() {
    // retrieve settings, if settings not set, save default
    if(false === get_option('example_theme_options', example_get_default_theme_options()))
        add_option('example_theme_options', example_get_default_theme_options());

    //group name (can be any, see settings_fields() call in example_theme_options_render_page()), option name (look at add_option, get_option), validate function callback
    register_setting('example_options', 'example_theme_options', 'example_theme_options_validate');

    // id, title, render callback function, url slug
    add_settings_section('general', '', '__return_false', 'theme_options');
    // $options[KEY], label, render callback function, url slug, settings_section id
    add_settings_field('slogan', 'Slogan label', 'example_settings_field_slogan', 'theme_options', 'general');
    add_settings_field('contacts', 'Contacts label', 'example_settings_field_contacts', 'theme_options', 'general');
    add_settings_field('copyright', 'Copy label', 'example_settings_field_copyright', 'theme_options', 'general');
    add_settings_field('logotype', 'Logotype label', 'example_settings_field_logotype', 'theme_options', 'general');
}
add_action('admin_init', 'example_settings_api_init');

// render for $options['slogan'] field
function example_settings_field_slogan() { $options = get_option('example_theme_options', example_get_default_theme_options()); ?>
    <input type="text" name="example_theme_options[slogan]" id="slogan" value="<?php echo esc_attr($options['slogan']); ?>" />
<?php }

function example_settings_field_contacts() { $options = get_option('example_theme_options', example_get_default_theme_options()); ?>
    <?php the_editor($options['contacts'], 'example_theme_options[contacts]', '', true, 2); ?>
<?php }

function example_settings_field_copyright() { $options = get_option('example_theme_options', example_get_default_theme_options()); ?>
    <?php the_editor($options['copyright'], 'example_theme_options[copyright]', '', true, 3); ?>
<?php }

function example_settings_field_logotype() { $options = get_option('example_theme_options', example_get_default_theme_options()); ?>
    <?php if(!empty($options['logotype'])):?>
        <div>
            <img class="upload_preview" src="<?php echo esc_attr($options['logotype'])?>" />
        </div>
    <?php endif;?>
    <input type="text" class="upload_field" name="example_theme_options[logotype]" id="logotype" value="<?php echo esc_attr($options['logotype']); ?>" />
    <input type="button" class="button upload_button" value="Upload" />
    <input type="button" class="button upload_clear" value="Remove" />
<?php }

// validation callback, gets array of $input, must retrieve the same array of $output, which will be saved to options
function example_theme_options_validate($input) {
    $output = $defaults = example_get_default_theme_options();

    $output['slogan'] = empty($input['slogan']) ? $defaults['slogan'] : $input['slogan'];
    $output['contacts'] = empty($input['contacts']) ? $defaults['contacts'] : $input['contacts'];
    $output['copyright'] = empty($input['copyright']) ? $defaults['copyright'] : $input['copyright'];
    $output['logotype'] = empty($input['logotype']) ? $defaults['logotype'] : $input['logotype'];

    if(!empty($output['logotype'])) { // allow only jpg, png and gif logotypes
        $output['logotype'] = in_array(strtolower(end(explode('.', $output['logotype']))), array('jpg', 'jpeg', 'png', 'gif')) ? $output['logotype'] : '';
    }

    if(!empty($output['logotype'])) { // try to get full image
        if(0 === strpos($output['logotype'], home_url())) { // if it is local image
            if(preg_match('/(.*?)\-\d+x\d+\.(jpg|jpeg|gif|png)$/usi', $output['logotype'], $match)) { //not full size
                $upload_dir = wp_upload_dir();

                if(file_exists($upload_dir['basedir'] . DIRECTORY_SEPARATOR . str_replace($upload_dir['baseurl'], '', $match[1] . '.' . $match[2]))) {
                    $output['logotype'] = $match[1] . '.' . $match[2];
                }
            }
        }
    }

    return $output;
}

// render page callback
function example_theme_options_render_page() { ?>
    <div class="wrap">
        <?php screen_icon(); ?>
        <h2>Theme options</h2>
        <?php settings_errors(); ?>

        <form method="post" action="options.php">
            <?php
                settings_fields('example_options');
                do_settings_sections('theme_options');
                submit_button();
            ?>
        </form>
    </div>
<?php }
```

To display option in theme use helper functions like so:

```php
<?php echo apply_filters('the_content', get_swickd_option('copyright')) ?>
```
