---
layout: post
title: Self signed ssl for nginx
tags: [ssl, nginx]
---

Create sertificates
-------------------

```sh
openssl req -new -x509 -days 365 -nodes -out /home/mac/nginx.pem -keyout /home/mac/nginx.key
```

You may create 2048-bit certificate by adding `-newkey rsa:2048` to the previous command like this:

```sh
openssl req -new -x509 -days 365 -nodes -out /srv/ssl/nginx.pem -keyout /srv/ssl/nginx.key -newkey rsa:2048
```

Openssl will ask you few questions:

```
Cuntry Name: UK
State or Province Name: [Empty]
Locality Name: Kiev
Organization Name: mac
Organization Unit Name: Web Services
Common Name: example.com # <- MUST BE YOUR DOMAIN
Email Address: marchenko.alexandr@gmail.com
```

Configure nginx
---------------

Example of my config:

```
server {
    listen 80;
    rewrite ^(.*) https://$host$1 permanent; # redirect all to https
    root /home/mac/www;
    index index.html;
    server_name localhost;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
server {
    listen 443 default_server ssl;
    root /home/mac/www;
    index index.html;
    server_name localhost;

    ssl_certificate /home/mac/nginx.pem;
    ssl_certificate_key /home/mac/nginx.key;
    ssl_session_timeout 10m;
    keepalive_timeout 70;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Links:

http://library.linode.com/web-servers/nginx/configuration/ssl
http://wiki.nginx.org/HttpSslModule
