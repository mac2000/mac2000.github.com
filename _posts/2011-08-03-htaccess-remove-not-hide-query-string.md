---
layout: post
title: htaccess remove (not hide) query string

tags: [htaccess, mod_rewrite, query_string, request_uri, rewrite, rewritecond, rewriterule]
---

```
RewriteEngine On

RewriteCond %{REQUEST_URI} ^/index.php$ [NC,OR]
#RewriteCond %{REQUEST_URI} ^/index.html$ [NC,OR]
#RewriteCond %{REQUEST_URI} ^/index.htm$ [NC,OR]
#RewriteCond %{REQUEST_URI} ^/index$ [NC,OR]
#RewriteCond %{REQUEST_URI} ^/home$ [NC,OR]
RewriteCond %{REQUEST_URI} ^/$
RewriteCond %{QUERY_STRING} .
RewriteCond %{QUERY_STRING} !q=
RewriteRule .* /? [R=301,L]
```

This will redirect all requests like:

http://example.com/?a=b

http://example.com/index.php?a=b

to http://example.com

but will not redirect thouse requests that contains q=, this may be changed to whatever your system uses for human readable urls, wordpress uses p=

in my case, sustem transparently redirect requests like:

http://example.com/home to http://example.com/index.php?q=/home and we will not touch such requests

http://wiki.apache.org/httpd/RewriteQueryString
