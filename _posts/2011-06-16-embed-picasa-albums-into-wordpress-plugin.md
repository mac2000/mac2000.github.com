---
layout: post
title: Embed picasa albums into wordpress plugin

tags: [gdata, picasa, plugin, wordpress, zend]
---

Plugin allows insert picasa albums into posts and pages via new tinymce button in few clicks.

Used build in jquery ui and zend gdata library.

![screenshot](/images/wp/screenshot-1.png) ![screenshot](/images/wp/screenshot-2.png) ![screenshot](/images/wp/screenshot-3.png)

plugin code:

```php
<?php
/*
Plugin Name: Embed picasa album
Plugin URI: http://wordpress.org/
Description: Embed picasa album into post or page.
Author: Marchenko Alexandr
Version: 1.1
Author URI: http://mac-blog.org.ua/
*/

//http://www.presscoders.com/wordpress-settings-api-explained/

/////////////////////////////////////////////////////////////////////
//add plugin options page
add_action( 'admin_menu', 'embpicasa_admin_menu' );
function embpicasa_admin_menu() {
    add_options_page('Picasa settings', 'Picasa', 'manage_options', __FILE__, 'embpicasa_settings_page');
}
function embpicasa_settings_page() {
?>
    <div class="wrap">
        <div class="icon32" id="icon-options-general"><br></div>
        <h2>Picasa settings</h2>
        Enter auth params and select preferred image dimensions
        <form action="options.php" method="post">
        <?php settings_fields('embpicasa_options'); ?>
        <?php do_settings_sections(__FILE__); ?>
        <p class="submit">
            <input name="Submit" type="submit" class="button-primary" value="<?php esc_attr_e('Save Changes'); ?>" />
        </p>
        </form>
    </div>
<?php
}

/////////////////////////////////////////////////////////////////////
//register plugin options
add_action('admin_init', 'embpicasa_admin_init' );
function embpicasa_admin_init(){
    register_setting('embpicasa_options', 'embpicasa_options', 'embpicasa_options_validate' ); // group, name in db, validation func

    add_settings_section('auth_section', 'Auth Settings', 'embpicasa_options_section_auth', __FILE__);
    add_settings_field('embpicasa_options_login', 'Login', 'embpicasa_options_login_field_renderer', __FILE__, 'auth_section');
    add_settings_field('embpicasa_options_password', 'Password', 'embpicasa_options_password_field_renderer', __FILE__, 'auth_section');

    add_settings_section('img_section', 'Image Settings', 'embpicasa_options_section_img', __FILE__);
    add_settings_field('embpicasa_options_thumb_size', 'Thumbnail size', 'embpicasa_options_thumb_size_field_renderer', __FILE__, 'img_section');
    add_settings_field('embpicasa_options_full_size', 'Full image size', 'embpicasa_options_full_size_field_renderer', __FILE__, 'img_section');
    add_settings_field('embpicasa_options_crop', 'Crop images', 'embpicasa_options_crop_field_renderer', __FILE__, 'img_section');
}

function embpicasa_options_section_auth() {
    echo '<p>Your login and password in picasa</p>';
}

function embpicasa_options_section_img() {
    echo '<p>Preferred image dimensions</p>';
}

function embpicasa_options_login_field_renderer() {
    $options = get_option('embpicasa_options');
    echo "<input id='embpicasa_options_login' name='embpicasa_options[embpicasa_options_login]' size='40' type='text' value='{$options['embpicasa_options_login']}' />";
}

function embpicasa_options_password_field_renderer() {
    $options = get_option('embpicasa_options');
    echo "<input id='embpicasa_options_password' name='embpicasa_options[embpicasa_options_password]' size='40' type='password' value='{$options['embpicasa_options_password']}' />";
}

function embpicasa_options_thumb_size_field_renderer() {
    $options = get_option('embpicasa_options');
    $items = array('32', '48', '64', '72', '104', '144', '150', '160');
    echo "<select id='embpicasa_options_thumb_size' name='embpicasa_options[embpicasa_options_thumb_size]'>";
    foreach($items as $item) {
        $selected = ($options['embpicasa_options_thumb_size']==$item) ? 'selected="selected"' : '';
        echo "<option value='$item' $selected>$item</option>";
    }
    echo "</select>";
}

function embpicasa_options_full_size_field_renderer() {
    $options = get_option('embpicasa_options');
    $items = array('94', '110', '128', '200', '220', '288', '320', '400', '512', '576', '640', '720', '800', '912', '1024', '1152', '1280', '1440', '1600');
    echo "<select id='embpicasa_options_full_size' name='embpicasa_options[embpicasa_options_full_size]'>";
    foreach($items as $item) {
        $selected = ($options['embpicasa_options_full_size']==$item) ? 'selected="selected"' : '';
        echo "<option value='$item' $selected>$item</option>";
    }
    echo "</select>";
}

function embpicasa_options_crop_field_renderer() {
    $options = get_option('embpicasa_options');
    $items = array('no', 'yes');
    echo "<select id='embpicasa_options_crop' name='embpicasa_options[embpicasa_options_crop]'>";
    foreach($items as $item) {
        $selected = ($options['embpicasa_options_crop']==$item) ? 'selected="selected"' : '';
        echo "<option value='$item' $selected>$item</option>";
    }
    echo "</select>";
}

function embpicasa_options_validate($input) {
    // strip all fields
    $input['embpicasa_options_login']      =  wp_filter_nohtml_kses($input['embpicasa_options_login']);
    $input['embpicasa_options_password']   =  wp_filter_nohtml_kses($input['embpicasa_options_password']);
    $input['embpicasa_options_thumb_size'] =  wp_filter_nohtml_kses($input['embpicasa_options_thumb_size']);
    $input['embpicasa_options_full_size']  =  wp_filter_nohtml_kses($input['embpicasa_options_full_size']);

    // check image dimensions
    $items = array('32', '48', '64', '72', '104', '144', '150', '160');
    if(!in_array($input['embpicasa_options_thumb_size'], $items)) {
        $input['embpicasa_options_thumb_size'] = '150';
    }

    $items = array('32', '48', '64', '72', '104', '144', '150', '160');
    if(!in_array($input['embpicasa_options_full_size'], $items)) {
        $input['embpicasa_options_full_size'] = '640';
    }

    return $input;
}

// Define default option settings
register_activation_hook(__FILE__, 'embpicasa_options_add_defaults');
function embpicasa_options_add_defaults() {
    update_option('embpicasa_options', array(
        'embpicasa_options_login'      => 'LOGIN@gmail.com',
        'embpicasa_options_password'   => '',
        'embpicasa_options_thumb_size' => '150',
        'embpicasa_options_full_size'  => '640',
        'embpicasa_options_crop'       => 'no'
    ));
}

/////////////////////////////////////////////////////////////////////
// add the shortcode handler for picasa galleries
// http://brettterpstra.com/adding-a-tinymce-button/
function add_embpicasa_shortcode($atts, $content = null) {
        extract(shortcode_atts(array( "id" => '' ), $atts));

        if(empty($id)) return '';

        $options = get_option('embpicasa_options');

        if(!empty($options['embpicasa_options_login']) && !empty($options['embpicasa_options_password'])) {
            try {
                set_include_path(implode(PATH_SEPARATOR, array(
                    realpath(dirname(__FILE__) . '/library'),
                    get_include_path(),
                )));

                require_once 'Zend/Loader.php';

                Zend_Loader::loadClass('Zend_Gdata');
                Zend_Loader::loadClass('Zend_Gdata_Query');
                Zend_Loader::loadClass('Zend_Gdata_ClientLogin');
                Zend_Loader::loadClass('Zend_Gdata_Photos');
                Zend_Loader::loadClass('Zend_Gdata_Photos_UserQuery');
                Zend_Loader::loadClass('Zend_Gdata_Photos_AlbumQuery');
                Zend_Loader::loadClass('Zend_Gdata_Photos_PhotoQuery');

                $client = Zend_Gdata_ClientLogin::getHttpClient($options['embpicasa_options_login'], $options['embpicasa_options_password'], Zend_Gdata_Photos::AUTH_SERVICE_NAME);
                $service = new Zend_Gdata_Photos($client);

                $photos = array();
                $query = new Zend_Gdata_Photos_AlbumQuery();
                $query->setAlbumId($id);
                // http://code.google.com/intl/ru/apis/picasaweb/docs/1.0/reference.html
                $suffix = $options['embpicasa_options_crop'] == 'no' ? 'u' : 'c';
                $query->setThumbsize($options['embpicasa_options_thumb_size'] . $suffix);
                $query->setImgMax($options['embpicasa_options_full_size'] . $suffix);
                $results = $service->getAlbumFeed($query);
                while($results != null) {
                    foreach($results as $entry) {
                        foreach($results as $photo) {
                            $photos[] = array(
                                'thumbnail' => $photo->mediaGroup->thumbnail[0]->url,
                                'fullsize' => $photo->mediaGroup->content[0]->url
                            );
                        }
                    }
                    try {
                        $results = $results->getNextFeed();
                    }
                    catch(Exception $e) {$results = null;}
                }

                //TODO: here is theming, change it as u need

                $html = '<ul class="embpicasa">';

                foreach($photos as $photo) {
                    $html = $html . '<li>';
                    $html = $html . '<a rel="lightbox[' . $album['id'] . ']" target="_blank" href="' . $photo['fullsize'] . '">';
                    $html = $html . '<img src="' . $photo['thumbnail'] . '" />';
                    $html = $html . '</a>';
                    $html = $html . '</li>';
                    $opts = $opts . '<option value="' . $album['id'] . '">' . $album['name'] . '</option>';
                }
                $html = $html . '</ul>';

                //$html = $html . '<style type="text/css">';
                //$html = $html . '.embpicasa li {width:' . $options['embpicasa_options_thumb_size'] . 'px;height:' . $options['embpicasa_options_thumb_size'] . 'px;}';
                //$html = $html . '</style>';

                return $html;

            } catch(Exception $ex) {
                return '<p style="color:red">' . $ex->getMessage() . '</p>';
            }
        } else {
            return ''; //empty login or password
        }

        //TODO: here will be all zend gdata stufs to retrive album photos
        return '<p style="text-align:center">'.$id.'</p>';
}
add_shortcode('embpicasa', 'add_embpicasa_shortcode');

/////////////////////////////////////////////////////////////////////
// embed some javascript for tinymce plugin

// add jquery ui styles
function embpicasa_init() {
    if(is_admin() && current_user_can('edit_posts') && current_user_can('edit_pages') && get_user_option('rich_editing') == 'true') {
        wp_register_style( 'jquery-style', 'http://ajax.googleapis.com/ajax/libs/jqueryui/1.8.1/themes/smoothness/jquery-ui.css', true);
        wp_enqueue_style( 'jquery-style' );
    }

    if(!is_admin()) {
        wp_register_style( 'embpicasa-style', plugins_url('embpicasa.css', __FILE__));
        wp_enqueue_style( 'embpicasa-style' );
    }
}
add_action( 'init', 'embpicasa_init' );

add_action( 'edit_form_advanced', 'embpicasa_embed_js' );
add_action( 'edit_page_form', 'embpicasa_embed_js' );
function embpicasa_embed_js() {
?>
<script type="text/javascript">
    function embpicasa_dlg_open() {
        embpicasa_dlg_close();

        jQuery("#embpicasa_dlg").dialog({
            modal: true,
            draggable: false,
            resizable: false,
            buttons: {'Insert': embpicasa_dlg_insert}
        });

        jQuery("#embpicasa_dlg").dialog("open");
    }

    function embpicasa_dlg_insert() {
        var album_id = jQuery("#embpicasa_dlg_content_album").val();

        var shortcode = '[embpicasa id="'+album_id+'"]';

        if ( typeof tinyMCE != 'undefined' && ( ed = tinyMCE.activeEditor ) && !ed.isHidden() ) {
            ed.focus();
            if (tinymce.isIE)
                ed.selection.moveToBookmark(tinymce.EditorManager.activeEditor.windowManager.bookmark);

            ed.execCommand('mceInsertContent', false, shortcode);
        } else
            edInsertContent(edCanvas, shortcode);

        embpicasa_dlg_close();
    }

    function embpicasa_dlg_close() {
        jQuery("#embpicasa_dlg").dialog("close");
    }
</script>

<?php
}

function embpicasa_js_dlg_markup() {
$options = get_option('embpicasa_options');
$success = true;
$msg = '';
$opts = '';

if(!empty($options['embpicasa_options_login']) && !empty($options['embpicasa_options_password'])) {
    try {
        set_include_path(implode(PATH_SEPARATOR, array(
            realpath(dirname(__FILE__) . '/library'),
            get_include_path(),
        )));

        require_once 'Zend/Loader.php';

        Zend_Loader::loadClass('Zend_Gdata');
        Zend_Loader::loadClass('Zend_Gdata_Query');
        Zend_Loader::loadClass('Zend_Gdata_ClientLogin');
        Zend_Loader::loadClass('Zend_Gdata_Photos');
        Zend_Loader::loadClass('Zend_Gdata_Photos_UserQuery');
        Zend_Loader::loadClass('Zend_Gdata_Photos_AlbumQuery');
        Zend_Loader::loadClass('Zend_Gdata_Photos_PhotoQuery');

        $client = Zend_Gdata_ClientLogin::getHttpClient($options['embpicasa_options_login'], $options['embpicasa_options_password'], Zend_Gdata_Photos::AUTH_SERVICE_NAME);
        $service = new Zend_Gdata_Photos($client);

        $albums = array();

        $results = $service->getUserFeed();
        while($results != null) {
            foreach($results as $entry) {
                $album_id = $entry->gphotoId->text;
                $album_name = $entry->title->text;

                $albums[] = array(
                    'id' => $album_id,
                    'name' => $album_name
                );
            }

            try {
                $results = $results->getNextFeed();
            }
            catch(Exception $e) {$results = null;}
        }

        foreach($albums as $album) {
            $opts = $opts . '<option value="' . $album['id'] . '">' . $album['name'] . '</option>';
        }

    } catch(Exception $ex) {
        $success = false;
        $msg = $ex->getMessage();
    }
}
?>
<div class="hidden">
    <div id="embpicasa_dlg" title="Picasa">
        <div class="embpicasa_dlg_content">
            <?php if($success):?>
                <p>
                    <label>
                        Select album:
                        <select id="embpicasa_dlg_content_album" style="width:98%"><?php echo $opts;?></select>
                    </label>
                </p>
            <?php else:?>
                <div style="padding:1em;" class="ui-state-error ui-corner-all">
                    <p><strong>ERROR</strong><br /><?php echo $msg?></p>
                </div>
            <?php endif;?>
        </div>
    </div>
</div>
<style type="text/css">
.ui-button-text-only .ui-button-text {padding:0;}
.ui-widget-overlay {background:#AAAAAA;}
</style>
<?php
}
add_action( 'admin_footer', 'embpicasa_js_dlg_markup' );

/////////////////////////////////////////////////////////////////////
// add embpicasa button into tinymce
// http://brettterpstra.com/adding-a-tinymce-button/
function add_embpicasa_button() {
    if ( ! current_user_can('edit_posts') && ! current_user_can('edit_pages') )
        return;
    if ( get_user_option('rich_editing') == 'true') {
        add_filter('mce_external_plugins', 'add_embpicasa_tinymce_plugin');
        add_filter('mce_buttons', 'register_embpicasa_button');
    }
}
add_action('init', 'add_embpicasa_button');

function register_embpicasa_button($buttons) {
    array_push($buttons, "|", "embpicasa");
    return $buttons;
}

function add_embpicasa_tinymce_plugin($plugin_array) {
    $plugin_array['embpicasa'] = WP_PLUGIN_URL.'/'.str_replace(basename( __FILE__),"",plugin_basename(__FILE__)) . 'embpicasa.js';
    return $plugin_array;
}

// Trick/hack to tinymce refresh all files
function embpicasa_refresh_mce($ver) {
    $ver += 3;
    return $ver;
} add_filter( 'tiny_mce_version', 'embpicasa_refresh_mce');
```

Here u can see how to make options for plugins and pages to set them, also there is default values for them.

Then how to make shortcodes. In shortcode u may see how to retrive picasa album photos.

And last bit of code makes jquery ui dialog and adds tinymce button.

Here is tinymce plugin code:

```js
(function() {
    tinymce.create('tinymce.plugins.embpicasa', {
        init : function(ed, url) {
            if ( typeof embpicasa_dlg_open == 'undefined' ) return;

            ed.addButton('embpicasa', {
                title : 'Picasa',
                image : url+'/embpicasa.gif',
                onclick : function() {
                    embpicasa_dlg_open();
                }
            });
        },
        createControl : function(n, cm) {
            return null;
        },
        getInfo : function() {
            return {
                longname : "Embed picasa album",
                author : 'Marchenko Alexandr',
                authorurl : 'http://mac-blog.org.ua/',
                infourl : 'http://mac-blog.org.ua/',
                version : "1.0"
            };
        }
    });
    tinymce.PluginManager.add('embpicasa', tinymce.plugins.embpicasa);
})();
```

http://code.google.com/p/embpicasa/
