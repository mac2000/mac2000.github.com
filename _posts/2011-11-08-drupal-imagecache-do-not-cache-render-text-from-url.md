---
layout: post
title: Drupal imagecache do not cache, render text from url

tags: [batch, cache, drupal, gd, imagecache, imagecache_external, imagemagick, processing, resize]
---

Changes to **imagecache.module**

```php
...
function imagecache_cache() {
    $args = func_get_args();
    $preset = check_plain(array_shift($args));
    $path = implode('/', $args);

//start
//delete file if it already exists
$dst = imagecache_create_path($preset, $path);
$root_path=realpath(drupal_get_path('module', 'node').'/../../');
@unlink($root_path . '/' . $dst);
//end

    _imagecache_cache($preset, $path);

}
...
```

Changes to drupal **.htaccess**

```
...
#start
#all request to image cache must be proccessed with drupal
#no matter is file already exists
RewriteCond %{REQUEST_URI} ^.*/imagecache/.*/.*$
RewriteRule ^(.*)$ index.php?q=$1 [L,QSA]
#end

    # Rewrite URLs of the form 'x' to the form 'index.php?q=x'.
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_URI} !=/favicon.ico
    RewriteRule ^(.*)$ index.php?q=$1 [L,QSA]
...
```

Now every request to imagecache will be processed again and again, try to add **Add Text** action with following params:

```php
return isset($_REQUEST['t']) ? $_REQUEST['t'] : 'No text'
```

Do not forget to check **Evaluate Text as PHP code**

Now you can open urls like:

```
http://drupalimage.local/sites/default/files/imagecache/preset1/imagecache_sample.png?t=hello
http://drupalimage.local/sites/default/files/imagecache/preset1/imagecache_sample.png?t=world
```

Each time you will get new image with new text.

Now time for **imagecache_external** module, **version 1, not 2**. With it you will be able to do same things to external images, just make some changes to url like this:

```
http://drupalimage.local/external/preset1/http://shtirlitz.com/wp-content/uploads/2011/03/vishenki1.jpg?t=hello
```

Notice: Question marks (?) in image url must be replaced to %3F
