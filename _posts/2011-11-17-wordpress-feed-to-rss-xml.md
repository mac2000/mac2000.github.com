---
layout: post
title: WordPress /feed to rss.xml

tags: [301, 302, feed, feedburner, htaccess, redirect, rewrite, rss, wordpress]
---

```
<IfModule mod_rewrite.c>

RewriteEngine On
RewriteBase /
RewriteRule ^atom.xml$ ?feed=atom [R=302,L]
RewriteRule ^rss.xml$ ?feed=rss2 [R=302,L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
```
