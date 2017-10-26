---
layout: post
title: WordPress multisite

tags: [farm, multisite, wordpress, wp, wpmu]
---

Необходимо реализовать ферму сайтов на wordpress.

Для начала, необходим главный домен, который будет смотреть на наш сервер, а так же резолвить любые поддомены на него же.

В моем случае это домен multi.com

Если домен уже где то припаркова просто добавляем запись:

```
* A 192.168.5.11
```

Если домен у нас - необходимо поставить и настроить DNS.

**/etc/bind/named.conf.local**:

```
zone "multi.com" {
        type master;
        file "/etc/bind/db.multi.com";
};
```

**/etc/bind/db.multi.com**:

```
$TTL    604800
@       IN      SOA     multi.com. root.multi.com. (
                                2         ; Serial
                            604800         ; Refresh
                            86400         ; Retry
                        2419200         ; Expire
                            604800 )       ; Negative Cache TTL
;
@       IN      NS      multi.com.
@       IN      A       192.168.5.11
@       IN      AAAA    ::1
*.multi.com. IN A 192.168.5.11
```

Теперь настройка apache. Во первых надо убедиться что включен `mod_rewrite`

```
sudo a2enmod rewrite
```

Добавляем и включаем сайты

**/etc/apache2/sites-available/default**:

````
<VirtualHost *:80>
        ServerName multi.com
        ServerAlias *.multi.com
        DocumentRoot /var/www
        <Directory /var/www/>
                Options All
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>
</VirtualHost>
```

**/etc/apache2/sites-available/test1.ru**:

```
<VirtualHost *:80>
        ServerName test1.ru
        ServerAlias www.test1.ru
</VirtualHost>
```

**/etc/apache2/sites-available/test2.ru**:

```
<VirtualHost *:80>
        ServerName test2.ru
        ServerAlias www.test2.ru
</VirtualHost>
```

Включаем сайты:

```
a2ensite test1.ru
a2ensite test2.ru
```

Теперь файлы в /var/www должны быть доступны с доменов `multi.com`, `*.multi.com`, `test1.ru`, `test2.ru.`

Эти шаги важны так как multi site не будет работать на локалхосте.

Устанавливаем Wordpress, как обычный блог.

После чего, в файл **wp-config.php**, в самое начало, добавляем:

```php
define('WP_ALLOW_MULTISITE', true);
```

Теперь в админке в разделе **Инструменты** должен появиться новый пункт **Установка сети**.

![screenshot](/images/wp/114.png)

Заходим туда, и в разделе **Адреса сайтов вашей сети** выбираем **Поддомены**.

![screenshot](/images/wp/26.png)

Давим кнопку **Установить**. Wordpress матюкнеться что не может достучаться до поддоменов - на это можно не обращать внимания.

Далее следуем инструкциям wordpress'а (нужно внести изменения в файл **wp-config.php** и **.htaccess** и создать папку **blogs.dir**)

После внесения всех изменений необходимо перелогиниться в админку.

Если все было сделано правильно в разделе **Консоль** появиться новый пункт **Мои сайты**.

![screenshot](/images/wp/32.png)

Управление сетью находиться по адресу: **/wp-admin/network/** собственно туда и нужно идти чтобы добавить новые сайты. В самом верхнем-правом углу в выпадайке - там где пишут "Привет, admin"

![screenshot](/images/wp/6.png)

![screenshot](/images/wp/41.png)

При добавлении нового сайта - указываем его имя (не домен), заголовок и email админа (вбивать тот же email что и для админа - чтобы не плодить пользователей).

В моем случае:

![screenshot](/images/wp/71.png)

Теперь этот сайт доступен по адресу: **test1.multi.com**, можно в него залогиниться под тем же логином паролем что и для основного сайта (это возможно только при условии что добавляя новый сайт мы указали email администратора основного сайта, иначе будет создан отдельный пользователь для сайта).

Чтобы сделать его доступным по адресу **test1.ru**, необходим модуль:

http://wordpress.org/extend/plugins/wordpress-mu-domain-mapping/

Качаем разорхивируем, копируем файл `domain_mapping.php` в `/wp-content/plugins`, файл `sunrise.php` - в `/wp-content`.

В самый конец файла `wp-config.php` перед последним `require_once` добавляем, чтобы получилось так:

```php
define( 'SUNRISE', 'on' );

/** Инициализирует переменные WordPress и подключает файлы. */
require_once(ABSPATH . 'wp-settings.php');
```

После чего идем в плагины и активируем для сети плагин. (Под "активацией для сети" подразумевается активация модуля на странице `/wp-admin/network/plugins.php`. Плагины активированные тут будут автоматически включены для всех сайтов сети, при этом администратор сайта может устанавливать и удалять свои модули но не может удалить уже установленный модуль.)

Далее переходим на страницу настроек плагина `/wp-admin/network/settings.php?page=dm_admin_page` и прописываем IP адрес нашего сервера.

Остальные настройки (галочки) я не трогал.

![screenshot](/images/wp/8.png)

Теперь идем на `test1.multi.com`, логинимся и идем в настроку доменов (`/wp-admin/tools.php?page=domainmapping`), прописываем в качестве домена `test1.ru`, и ставим галочку **Primary domain for this blog**.

![screenshot](/images/wp/9.png)

После сохранения изменений сайт будет доступен по адресу **test1.ru**.

При логине - перекидывает на test1.multi.com - это связано с cookies - иначе не получилось бы залогиниться - главное что фронэнд сайта на нужном домене.

К сожалению далеко не все модули можно включить сразу для всей сети. Но в любом случае это лучше чем необходимость каждый раз их качать и устанавливать.

Темы - ставяться как обычно в /wp-content/themes - после чего в настройках сети необходимо разрешить тему для выбора сайтами.

![screenshot](/images/wp/10.png)

Ссылки:

http://codex.wordpress.org/Create_A_Network

http://www.binaryturf.com/run-multiple-sites-wordpress-30-separate-domains/

Самое классное в этом всем - теперь:

* Нужно бекапить только одну базу и папку с файлами.
* У всех сайтов одинаковые логин\пароль к фтп, базе, админке.
* Процесс разворачивания нового сайта сводиться к его добавлению в админке и выбору темы.
