---
layout: post
title: Xampp alias

tags: [alias, apache, bind, link, ln, server, vhost, vhosts, virtualhost, xampp]
---

To add global aliases like phpmyadmin add them to `C:\Users\mac\xampp\apache\conf\extra\httpd-xampp.conf` at the end, like this:

```
#
# My GLOBAL aliases
#
<IfModule alias_module>
    # Alias for phpMyAdmin
    Alias "/pma" "C:\Users\mac\xampp\phpMyAdmin/"
    <Directory "C:\Users\mac\xampp\phpMyAdmin/">
        Options Indexes FollowSymlinks MultiViews Includes
            AllowOverride All
            Order allow,deny
            Allow from all
    </Directory>
</IfModule>
```

And if u want alias for specific host, add it to `C:\Users\mac\xampp\apache\conf\extra\httpd-vhosts.conf` like this:

```
<VirtualHost dumper.local:85>
    ServerName dumper.local
    DocumentRoot "C:\Users\mac\Dropbox\htdocs\dumper/"
    <Directory "C:\Users\mac\Dropbox\htdocs\dumper/">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>
    Alias "/pma2" "C:\Users\mac\xampp\phpMyAdmin/"
    <Directory "C:\Users\mac\xampp\phpMyAdmin/">
        Options Indexes FollowSymlinks MultiViews Includes
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```
