---
layout: post
title: Rewrite for static files first
tags: [rewrite, nginx, apache, iis, static, cache, html]
---

Usually we are catching any url though our `index.php`.

Also, usually, we are trying to cache our pages for faster response.

Also, we know that nginx is very, very fast for static files, so here is idea: why dont we cache our pages to static html files and serve them instead?

So our rewrite rules will try to find our cached html file first and then, if there is no such file, they will rewrite request to our backend.

Lets assume that we have following directory structure:

```
public_html/
├── cache/
│   ├── about/
│   │   └── contacts.html
│   └── about.html
└── index.php
```

If user requests `/about` - we will show him cached `about.html` version, if user will request `/about/contacts` we will also show him cached `contacts.html`, but if user will ask something that still not cached we will ask `index.php` to do the work.

**nginx**

```
server {
    root /var/www;
    index index.php index.html index.htm;
    server_name localhost;

    # Notice `/cache$uri.html` - this is the trick
    location / {
        try_files /cache$uri.html $uri $uri/ @rewrite;
    }

    location @rewrite {
        rewrite ^/(.*)$ /index.php?q=$1;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }
}
```

**Apache**

```
RewriteEngine On

# Our cache
RewriteCond %{DOCUMENT_ROOT}/cache/$1.html -f
RewriteRule ^(.*)$ /cache/$1.html [QSA,L]

# Usual case
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ /index.php?q=$1 [QSA,L]
```

**IIS**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="Static cache" patternSyntax="Wildcard" stopProcessing="true">
                    <match url="*" />
                    <conditions>
                        <add input="{APPL_PHYSICAL_PATH}cache/{R:1}.html" matchType="IsFile" />
                    </conditions>
                    <action type="Rewrite" url="cache/{R:1}.html" />
                </rule>
                <rule name="Backend" patternSyntax="Wildcard" stopProcessing="true">
                    <match url="*" />
                    <conditions>
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php?q={R:1}" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

**PHP**

```php
<?php
ob_start();
echo $_GET['q'];
$html = ob_get_clean();
file_put_contents('cache/' . $_GET['q'] . '.html', $html);
echo $html;
```

Keep in mind that this is just example and never, never do like this!