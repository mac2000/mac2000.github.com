---
layout: post
title: mod_pagespeed filters
tags: [apache, mod_pagespeed, pagespeed, google, insights]
---

Google has developed mod_pagespeed module that is very easy to use and that acomplishes many Google Page Speed Insight recommendations.

Here is simple example how could you use it

Vagrantfile
-----------

```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.network "public_network"
    config.vm.provision :shell, path: "Provision.sh"
end
```

Provision
---------

```sh
#!/usr/bin/env bash

# Use closest mirrors available
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-updates main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-backports main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt trusty-security main restricted universe multiverse" | sudo tee --append /etc/apt/sources.list

sudo apt-get update
sudo apt-get install -y apache2

# https://developers.google.com/speed/pagespeed/module/download
wget https://dl-ssl.google.com/dl/linux/direct/mod-pagespeed-stable_current_amd64.deb
sudo dpkg -i mod-pagespeed-*.deb
sudo apt-get -f install
rm -rf mod-pagespeed-*.deb

sudo rm /etc/apache2/sites-enabled/000-default.conf
sudo ln -s /vagrant/Site.conf /etc/apache2/sites-enabled
sudo a2enmod headers setenvif mime rewrite authz_core deflate filter expires include
sudo service apache2 restart
```

Site config
-----------

```
<VirtualHost *:80>
    DocumentRoot /vagrant

    <Directory /vagrant>
        Options All
        AllowOverride All
        Require all granted
    </Directory>

    <Location /pagespeed>
        Order allow,deny
        #Allow from localhost
        #Allow from 127.0.0.1
        Allow from all #WARNING: this SHOULD be removed
        SetHandler pagespeed_admin
    </Location>
</VirtualHost>
```


Filter enabled by default
-------------------------

**General**:
[add_head](https://developers.google.com/speed/pagespeed/module/filter-head-add),
[extend_cache](https://developers.google.com/speed/pagespeed/module/filter-cache-extend)

**Images**:
[rewrite_images](https://developers.google.com/speed/pagespeed/module/filter-image-optimize)

**Styles**:
[combine_css](https://developers.google.com/speed/pagespeed/module/filter-css-combine),
[rewrite_css](https://developers.google.com/speed/pagespeed/module/filter-css-rewrite),
[flatten_css_imports](https://developers.google.com/speed/pagespeed/module/filter-flatten-css-imports),
[inline_css](https://developers.google.com/speed/pagespeed/module/filter-flatten-css-imports)

**Javascript**:
[rewrite_javascript](https://developers.google.com/speed/pagespeed/module/filter-js-minify),
[inline_javascript](https://developers.google.com/speed/pagespeed/module/filter-js-inline),
[combine_javascript](https://developers.google.com/speed/pagespeed/module/filter-js-combine)


Suggested (safe to use) filters
-------------------------------

**General**:
[collapse_whitespace](https://developers.google.com/speed/pagespeed/module/filter-whitespace-collapse),
[insert_dns_prefetch](https://developers.google.com/speed/pagespeed/module/filter-insert-dns-prefetch),
[remove_comments](https://developers.google.com/speed/pagespeed/module/filter-comment-remove),
[remove_quotes](https://developers.google.com/speed/pagespeed/module/filter-quote-remove),
[trim_urls](https://developers.google.com/speed/pagespeed/module/filter-trim-urls),
[elide_attributes](https://developers.google.com/speed/pagespeed/module/filter-attribute-elide),
[combine_heads](https://developers.google.com/speed/pagespeed/module/filter-head-combine)

**Images**:
[lazyload_images](https://developers.google.com/speed/pagespeed/module/filter-lazyload-images)

**Styles**:
[inline_google_font_css](https://developers.google.com/speed/pagespeed/module/filter-css-inline-google-fonts),
[move_css_to_head](https://developers.google.com/speed/pagespeed/module/filter-css-to-head),
[move_css_above_scripts](https://developers.google.com/speed/pagespeed/module/filter-css-above_scripts),
[prioritize_critical_css](https://developers.google.com/speed/pagespeed/module/filter-prioritize-critical-css)

**Scripts**:
[insert_ga](https://developers.google.com/speed/pagespeed/module/filter-insert-ga)


Project specific filters
------------------------

**Images**:
[resize_images](https://developers.google.com/speed/pagespeed/module/filter-image-optimize#resize_images),
[resize_rendered_image_dimensions](https://developers.google.com/speed/pagespeed/module/filter-image-optimize.html#resize_rendered_image_dimensions),
[sprite_images](https://developers.google.com/speed/pagespeed/module/filter-image-sprite),
[inline_preview_images](https://developers.google.com/speed/pagespeed/module/filter-inline-preview-images.html)


Magic (danger) filters
----------------------

**General**:
[local_storage_cache](https://developers.google.com/speed/pagespeed/module/filter-local-storage-cache)

**Javascript**:
[defer_javascript](https://developers.google.com/speed/pagespeed/module/filter-js-defer)


Ended up with following addition to [HTML 5 boilerplate .htaccess](https://raw.githubusercontent.com/h5bp/html5-boilerplate/master/dist/.htaccess):

```
<IfModule pagespeed_module>
    ModPagespeedEnableFilters collapse_whitespace,insert_dns_prefetch,remove_comments,remove_quotes,trim_urls,elide_attributes,combine_heads,lazyload_images,inline_google_font_css,move_css_to_head,move_css_above_scripts

    # prioritize_critical_css - only for big styles
    # insert_ga - only if you use Google Analytics
    # resize_images - project dependet, resizes: <img src="photo-1024x768.jpg" width="648" height="480">
    # resize_rendered_image_dimensions - project dependet, resize: <img src="photo-1024x768.jpg" style="max-width: 20em">
    # inline_preview_images - project dependet, inlines lowres images
    # sprite_images - project dependet, generates sprite map from css backgrounds
    # defer_javascript - magic
    # local_storage_cache - magic

    # ModPagespeedAnalyticsID UA-799647-3
</IfModule>
```

Here is all code: [mod_pagespeed.zip](/examples/mod_pagespeed.zip)

Clear (flush) mod_pagespeed cache
---------------------------------

```sh
sudo touch /var/cache/mod_pagespeed/cache.flush
```

After opening pages you can delete if

```
sudo rm -rf /var/cache/mod_pagespeed/cache.flush
```
