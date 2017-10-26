---
layout: post
title: XAMPP use Virtual Host but keep localhost

tags: [admin, administration, apache, config, lamp, server, vhost, vhosts, virtual, xampp]
---

`C:\xampp\apache\conf\extra\httpd-vhosts.conf`

```
NameVirtualHost 127.0.0.1:85

<VirtualHost 127.0.0.1:85>
    ServerName localhost
    DocumentRoot "C:/xampp/htdocs/"
</VirtualHost>

<VirtualHost 127.0.0.1:85>
    ServerName dumper.local
    DocumentRoot "C:/xampp/htdocs/dumper"
    <Directory "C:/xampp/htdocs/dumper">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```
