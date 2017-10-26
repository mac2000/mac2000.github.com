---
layout: post
title: Drupal wysiwyg CodeMirror

tags: [codemirror, d6, drupal, editarea, hook_editor, wysiwyg]
---

To add CodeMirror editor to wysiwyg drupal module, u need create following files:

**/sites/all/modules/wysiwyg/editors/codemirror.inc**

```php
<?php
/**
    * @file
    * Editor integration functions for CodeMirror.
    */

/**
    * Plugin implementation of hook_editor().
    */
function wysiwyg_codemirror_editor() {
    $editor['codemirror'] = array(
    'title' => 'CodeMirror',
    'vendor url' => 'http://codemirror.net',
    'download url' => 'http://codemirror.net/codemirror.zip',
    'library path' => wysiwyg_get_path('codemirror'),
    'libraries' => array(
        '' => array(
        'title' => 'CodeMirror',
        'files' => array('lib/codemirror.js'),
        ),
    ),
    'version callback' => 'wysiwyg_codemirror_version',
    'themes callback' => 'wysiwyg_codemirror_themes',
    'settings callback' => 'wysiwyg_codemirror_settings',
    'load callback' => 'wysiwyg_codemirror_load',
    'plugin callback' => 'wysiwyg_codemirror_plugins',
    'versions' => array(
        '2' => array(
        'js files' => array('codemirror.js'),
        'css files' => array('codemirror.css'),
        ),
    ),
    );
    return $editor;
}

/**
    * Detect editor version.
    *
    * @param $editor
    *   An array containing editor properties as returned from hook_editor().
    *
    * @return
    *   The installed editor version.
    */
function wysiwyg_codemirror_version($editor) {
    $fp = $editor['library path'] . '/README.md';
    if (!file_exists($fp)) {
        return;
    }
    $fp = fopen($fp, 'r');
    $line = fgets($fp);
    if (preg_match('@([0-9\.]+)$@', $line, $version)) {
        fclose($fp);
        return $version[1];
    }
    fclose($fp);
}

/**
    * Determine available editor themes or check/reset a given one.
    *
    * @param $editor
    *   A processed hook_editor() array of editor properties.
    * @param $profile
    *   A wysiwyg editor profile.
    *
    * @return
    *   An array of theme names. The first returned name should be the default
    *   theme name.
    */
function wysiwyg_codemirror_themes($editor, $profile) {
    $themes = array('default');
    //TODO: no need in this, i do not found way to use themes with wysiwyg module
    /*if ($handle = opendir($editor['library path'] . '/theme')) {
        while (false !== ($file = readdir($handle))) {
            if ($file != "." && $file != ".." && $file != "default.css") {
                if(pathinfo($file, PATHINFO_EXTENSION) == "css") {
                    $themes[] = pathinfo($file, PATHINFO_FILENAME);
                }
            }
        }
        closedir($handle);
    }*/
    return $themes;
}

/**
    * Perform additional actions upon loading this editor.
    *
    * @param $editor
    *   A processed hook_editor() array of editor properties.
    * @param $library
    *   The internal library name (array key) to use.
    */
function wysiwyg_codemirror_load($editor, $library) {
    drupal_add_css($editor['library path'] . '/lib/codemirror.css');
    drupal_add_css($editor['library path'] . '/theme/default.css'); //TODO: load apropriate theme file here, when wysiwyg will support this feature

    drupal_add_js($editor['library path'] . '/lib/codemirror.js');
    drupal_add_js($editor['library path'] . '/mode/xml/xml.js');
    drupal_add_js($editor['library path'] . '/mode/javascript/javascript.js');
    drupal_add_js($editor['library path'] . '/mode/css/css.js');
    drupal_add_js($editor['library path'] . '/mode/clike/clike.js');
    drupal_add_js($editor['library path'] . '/mode/php/php.js');
}

/**
    * Return runtime editor settings for a given wysiwyg profile.
    *
    * @param $editor
    *   A processed hook_editor() array of editor properties.
    * @param $config
    *   An array containing wysiwyg editor profile settings.
    * @param $theme
    *   The name of a theme/GUI/skin to use.
    *
    * @return
    *   A settings array to be populated in
    *   Drupal.settings.wysiwyg.configs.{editor}
    */
function wysiwyg_codemirror_settings($editor, $config, $theme) {
    $settings = array(
        'theme' => $theme,
    );

    if (isset($config['buttons']['default']['lineNumbers'])) {
        $settings['lineNumbers'] = (bool)$config['buttons']['default']['lineNumbers'];
    }
    if (isset($config['buttons']['default']['indentWithTabs'])) {
        $settings['indentWithTabs'] = (bool)$config['buttons']['default']['indentWithTabs'];
    }

    return $settings;
}

/**
    * Return internal plugins for this editor; semi-implementation of hook_wysiwyg_plugin().
    */
function wysiwyg_codemirror_plugins($editor) {
    $plugins = array(
    'default' => array(
        'buttons' => array(
        'lineNumbers' => t('Line numbers'),
        'indentWithTabs' => t('Indent with tabs'),
        ),
        'internal' => TRUE,
    ),
    );
    return $plugins;
}
```

**/sites/all/modules/wysiwyg/editors/js/codemirror.js**

```js
(function($) {

Drupal.wysiwyg.codemirror = {}; // CodeMirror instances storage

/**
 * Attach this editor to a target element.
 *
 * See Drupal.wysiwyg.editor.attach.none() for a full desciption of this hook.
 */
Drupal.wysiwyg.editor.attach.codemirror = function(context, params, settings) {
    if(typeof params != 'undefined') {
        Drupal.wysiwyg.codemirror[params.field] = CodeMirror.fromTextArea(document.getElementById(params.field), {
            lineNumbers: Drupal.settings.wysiwyg.configs.codemirror.format5.lineNumbers,//true,
            matchBrackets: true,
            mode: "application/x-httpd-php",
            indentUnit: 4,
            indentWithTabs: Drupal.settings.wysiwyg.configs.codemirror.format5.indentWithTabs,//true,
            enterMode: "keep",
            tabMode: "shift"
        });
    }
};

/**
 * Detach a single or all editors.
 *
 * See Drupal.wysiwyg.editor.detach.none() for a full desciption of this hook.
 */
Drupal.wysiwyg.editor.detach.codemirror = function(context, params) {
    if(typeof params != 'undefined') {
        Drupal.wysiwyg.codemirror[params.field].toTextArea();
        delete Drupal.wysiwyg.codemirror[params.field];
    }
};

})(jQuery);
```

**/sites/all/modules/wysiwyg/editors/css/codemirror.css**

```css
.CodeMirror {
    border: 1px solid #999;
}
```

## How to add custom editor for wysiyg

To add custom editor, u need create files as above.

Inc file must implement:

`wysiwyg_[editor]_editor` - this hook retrives info about editor, that will be used by wysiwyg module to determine is editor installed (found at /sites/all/libraries).

Versions sub array contains file names that must be included for specific versions (see for example tinymce.inc).

`wysiwyg_[editor]_version` - must return version of installed editor, used by previous function to include version specific files.

`wysiwyg_[editor]_themes` - actualy do nothing at moment, will be implemented in future versions of wysiwyg, must retrive array with at least one string (theme name).

`wysiwyg_[editor]_load` - optional call back that can be used to load additional files (look at yui.inc).

`wysiwyg_[editor]_settings` - return array of settings that will be encoded to json `Drupal.settings.wysiwyg.configs.[editor].format5` variable. Here u can access config variables that user entered on wysiwyg profile config page. All configs are the same for all editors, except editor plugins.

`wysiwyg_[editor]_plugins` - return array of checkboxes value label pairs, that will be shown on editor profile config page in plugins section.

Js file must implement:

`Drupal.wysiwyg.editor.attach.[editor]` - must attach editor to params.field.

`Drupal.wysiwyg.editor.detach.[editor]` - must detach editor from params.field.

Actualy this is it. Implementing this functions u can add any editor u want.

Commited patch to http://drupal.org/node/1214136 - but it was closed as duplicate to http://drupal.org/node/274431 that was created in 2008 O_o
