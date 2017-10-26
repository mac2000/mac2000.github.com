---
layout: post
title: Ubuntu configure Mass Virtual hosting

tags: [apache, apache2, host, hosting, mass, subdomain, ubuntu, vhost, virtualhost]
---

Task is to configure apache to dynamicaly use subdomains of existing folders

First of all we need to enable `vhost_alias module`

```sh
sudo a2enmod vhost_alias
```

Then describe our sites

```sh
sudo cp /etc/apache2/sites-available/default /etc/apache2/sites-available/mass
```

And make it look like this:

```
mac@x51rl:/$ cat /etc/apache2/sites-available/mass
<Directory /home/mac/Sites/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
</Directory>

<VirtualHost *:80>
    ServerName simple.x51rl.mam.org.ua
    ServerAlias *.simple.x51rl.mam.org.ua
    VirtualDocumentRoot /home/mac/Sites/simple/%0/
</VirtualHost>

<VirtualHost *:80>
        ServerName zf.x51rl.mam.org.ua
        ServerAlias *.zf.x51rl.mam.org.ua
        VirtualDocumentRoot /home/mac/Sites/zf/%0/public
</VirtualHost>
```

Now create `/home/mac/Sites/simple/test1.simple.x51rl.mam.org.ua/index.php` and it will be accessed on `test1.simple.x51rl.mam.org.ua`

Or for Zend example

`/home/mac/Sites/zf/test1.zf.x51rl.mam.org.ua/public/index.php` and it will be accessible on `test1.zf.x51rl.mam.org.ua`
