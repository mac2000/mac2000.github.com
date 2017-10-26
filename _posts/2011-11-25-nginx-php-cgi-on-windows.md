---
layout: post
title: nginx, php-cgi on windows

tags: [cgi, nginx, php, php-cgi, win, windows]
---

http://nginx.org/en/download.html

http://windows.php.net/download/

Нужен VC9 (6-й нужен только если используем с апачем).  Будет ли это thread safe или not thread safe значения не имеет.

Для запуска php-cgi.exe в фоновом режиме, использую [RunHiddenConsole](/images/wp/RunHiddenConsole.zip):

**start.cmd**

```sh
@ECHO OFF
start C:\nginx\nginx.exe
start C:\nginx\RunHiddenConsole.exe C:\PHP\php-cgi.exe -b 127.0.0.1:9000 -c C:\PHP\php.ini
EXIT
```

**stop.cmd**

```sh
@ECHO OFF
taskkill /f /IM nginx.exe
taskkill /f /IM php-cgi.exe
EXIT
```

http://eksith.wordpress.com/2008/12/08/nginx-php-on-windows/

http://wiki.nginx.org/PHPFastCGIOnWindows

**nginx.conf**

```
worker_processes  1;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    gzip  on;

    server {
        listen       85;
        server_name  localhost;
        charset utf-8;
        root   html;
        index  index.php index.html index.htm;

        location / {
            try_files $uri $uri/ /index.php?q=$uri&$args;
        }

        # wordpress example
        #location /blog {
        #   try_files $uri $uri/ /blog/index.php?q=$uri&$args;
        #}

        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

    }
}
```

Отличная статья на примере Wordpress'а показывающая как бороться с ЧПУ

http://wiki.nginx.org/WordPress

http://wiki.nginx.org/Drupal
