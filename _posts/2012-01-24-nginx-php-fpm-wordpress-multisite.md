---
layout: post
title: Nginx, php-fpm, wordpress multisite

tags: [fpm, multilingual, multisite, nginx, wordpress, wp]
---

```sh
apt-get install mysql-server mysql-client
apt-get install nginx
apt-get install php5-cli php5-common php5-mysql php5-suhosin php5-gd php5-dev php5-fpm php5-cgi php-pear php-apc
touch /etc/nginx/sites-available/example.com
ln -s /etc/nginx/sites-available/example.com
```

**/etc/nginx/sites-available/example.com**:

```
server {
    server_name *.example.com;
    index  index.php index.html index.htm;
    charset utf-8;
    root /var/www;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    rewrite /wp-admin$ $scheme://$host$uri/ permanent;

    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires max;
        log_not_found off;
    }

    rewrite /files/$ /index.php last;

    set $cachetest "$document_root/wp-content/ms-filemap/${host}${uri}";
    if ($uri ~ /$) {
        set $cachetest "";
    }
    if (-f $cachetest) {
        rewrite ^ /wp-content/ms-filemap/${host}${uri} break;
    }

    if ($uri !~ wp-content/plugins) {
        rewrite /files/(.+)$ /wp-includes/ms-files.php?file=$1 last;
    }

    if (!-e $request_filename) {
        rewrite ^/[_0-9a-zA-Z-]+(/wp-.*) $1 last;
        rewrite ^/[_0-9a-zA-Z-]+.*(/wp-admin/.*\.php)$ $1 last;
        rewrite ^/[_0-9a-zA-Z-]+(/.*\.php)$ $1 last;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

Пример взят от сюда: http://codex.wordpress.org/Nginx с той лиш разницей что этот будет работать из коробки.

После этого, все по [мануалке](/663/)


Добавление еще одного языка:

```sh
cd /var/www/wp-content/languages/
wget http://svn.automattic.com/wordpress-i18n/uk/tags/3.3.1/messages/uk.mo
cd /var/www/wp-content/themes/twentyeleven/languages/
wget http://svn.automattic.com/wordpress-i18n/uk/tags/3.3.1/messages/twentyeleven/uk.mo
```

Нужно не забыть выставить права 777 на папки uploads и blogs.dir

Создать базу для wordpress'а из консоли:

```sh
mysqladmin -u root -p -v create example
```

Идея была в том чтобы сделать многоязычный сайт на базе multisite.

Ссылки по теме:

http://codex.wordpress.org/Multilingual_WordPress

http://codex.wordpress.org/WordPress_in_Your_Language

http://svn.automattic.com/wordpress-i18n/

http://codex.wordpress.org/Installing_WordPress_in_Your_Language
