---
layout: post
title: Automatically add canonical header for urls with non empty query strings
tags: [htaccess, link, canonical, querystring]
---

Suppose we have simple html site and we still wish to use canonicals for our pages so Google can index them correct.

But here is tricky part - how could we automate adding canonicals for pages?

Seems that google undestands special `Link` header for canonicals - https://support.google.com/webmasters/answer/139066?hl=en#6

Here is what I end up with:

```
# If non empty query string add Canonical header
RewriteCond %{QUERY_STRING} ^.+$ [NC]
SetEnvIf Host (.*) HOSTNAME=$1
SetEnvIf Request_URI (.*) URI=$1
RewriteRule .* - [E=SET_CANONICAL_HEADER:1]
Header set Link "<http://%{HOSTNAME}e%{URI}e>; rel=\"canonical\"" env=SET_CANONICAL_HEADER
```

So we have non empty query string rewrite condition in first line.

Two next lines with `SetEnvIf` is needed just to duplicate apache variables into environment variables.

Our rewrite rule do nothing except setting `SET_CANONICAL_HEADER` into `1` only if rewrite condition met.

And if so we are setting header to exactly same url but without query string.



