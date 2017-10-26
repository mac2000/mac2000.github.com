---
layout: post
title: Ubuntu devenv

tags: [admin, administration, apache, apt, bind, dns, hosts, lamp, mass, postfix, tasksel, ubuntu, vhost]
---

Tips for configuring php development enviroment in ubuntu.

Installing apps
---------------

First of all install tasksel:

```sh
sudo apt-get install tasksel
```

To list available tasks use:

```sh
tasksel --list-tasks
```

We're going to install lamp-server, dns-server, mail-server.

Installing LAMP
---------------

We're going install apache, mysql and php, to do so, type:

```sh
sudo tasksel install lamp-server
```

This command will install and configure all this programs automaticaly. While installing you will be promted to enter root password for mysql.

After installing try to open http://localhost if all ok you will see message "It Works".

It is time to setup some packages that not installed by tasksel, like pear, gd, curl etc.

```sh
sudo apt-get install php-pear php5-xdebug php-apc curl libcurl3 php5-curl php5-gd php5-tidy php5-xmlrpc phpmyadmin cron wget phpunit libzend-framework-php libzend-framework-zendx-php zend-framework zend-framework-bin
```

While installing you will be promted for phpmyadmin password.

Also here is some examples using pear:

```sh
sudo pear install --alldeps PhpDocumentor
pear list
sudo pear upgrade-all
```

After all installed, last step to do is enable rewrite module (and some other modules), and you may host almost any site in /var/www, to do so type:

```sh
sudo a2enmod deflate
sudo a2enmod expires
sudo a2enmod rewrite
sudo /etc/init.d/apache2 restart
```

Now edit /etc/apache2/sites-available/default and change AllowOverride to All for / and /var/www directories, otherwise clean urls wont work. Do not forget restart apache after doing changes to configs.

Now you will be able to host probably anything in your /var/www. But I always need to host many projects, so i again and again create virtual host for each and record in /ets/hosts file and it makes me crazy. What i going to do is setup DNS server to resole any subdomain to localhost and configure apache mass virtual hosting.

**NOTICE:** to make `mod_rewrite` work u must add `RewriteBase /`

Installing DNS
--------------

To install DNS just type:

```sh
sudo tasksel install dns-server
```

I'm going to configure local.com domain, but it may be really anithing u want.

First edit /etc/bind/named.conf.local, and add:

```
zone "local.com" {
    type master;
    file "/etc/bind/db.local.com";
};
```

Then:

```sh
sudo cp /etc/bind/db.local /etc/bind/db.local.com
```

Then edit /etc/bind/db.local.com, replace all localhost string to local.com and add following line to end of file:

```
*.local.com. IN A 127.0.0.1
```

No check your file, restart bind and check is it works:

```sh
named-checkzone local.com /etc/bind/db.local.com
sudo /etc/init.d/bind9 restart
nslookup aa.bb.local.com 127.0.0.1
```

Now when DNS is working we need to tell dhcp to use it, just uncomment
following line in /etc/dhcp3/dhclient.conf:

```
prepend domain-name-servers 127.0.0.1;
```

Now after restart you will be able to ping any subdomain for local.com.

## Apache Mass Virtual Hosting

Main idea is that you will have folder projects, and its subfolders will be subdomains to local.com. To configure this you will need `vhost_alias` module, enable it by typing:

```sh
sudo a2enmod vhost_alias
sudo /etc/init.d/apache2 restart
```

Now create folders for your projects:

```sh
mkdir -p ~/Projects/zf
mkdir -p ~/Projects/drupal
mkdir -p ~/Projects/wp
mkdir -p ~/Projects/script
```

And create file /etc/apache2/sites-available/local.com and edit it like this:

```
# AllowOverride for mod_rewrite
# Do not forget to change directory path
<Directory /home/mac/Projects>
    Options All
    AllowOverride All
</Directory>

# Replacements for subdomains
# %0 - test1.drupal.local.com > test1.drupal.local.com
# %1 - test1.drupal.local.com > test1

<VirtualHost *:80>
        ServerAlias *.drupal.local.com
        VirtualDocumentRoot /home/mac/Projects/drupal/%1
</VirtualHost>

# If using netbeans
# ln -s /home/mac/NetBeansProjects /home/mac/Projects/nb
# Notice: project names must be in lower case to work
<VirtualHost *:80>
        ServerAlias *.nb.local.com
        VirtualDocumentRoot /home/mac/Projects/nb/%1
</VirtualHost>

<VirtualHost *:80>
        ServerAlias *.script.local.com
        VirtualDocumentRoot /home/mac/Projects/script/%1
</VirtualHost>

<VirtualHost *:80>
        ServerAlias *.wp.local.com
        VirtualDocumentRoot /home/mac/Projects/wp/%1
</VirtualHost>

<VirtualHost *:80>
    ServerAlias *.zf.local.com
    VirtualDocumentRoot /home/mac/Projects/zf/%1/public
</VirtualHost>
```

Now if you create ~/Projects/wp/test1/index.php it will be accessible on http://test1.wp.local.com.

All left to do is to configure mail.

Install mail server
-------------------

To install mail server just type:

```sh
sudo tasksel install mail-server
```

While installing mail server choose "Internet site". When you will be asked to enter domain type local.com.

After installing add following lines to end of /etc/postfix/main.cf file:

```ini
local_recipient_maps =
# Replace mac to your user name
luser_relay = mac
```

This lines tells that all mail will be mailed to specific user.

Now restart postfix:

```sh
sudo /etc/init.d/postfix restart
```

And you will be able to send mails to user1@local.com, user2@local.com etc.  All mail will be mailed to your local mailbox.

Try this script ~/Projects/script/mail/index.php, accessible via http://mail.script.local.com/, run it at least once when all done, to create local mailbox file:

```php
<?php
$r = mail('user1@local.com', 'test', 'test');
$m = $r ? 'yes' : 'no';
echo $m;
```

To retrive local mail, add account to evolution.

When evolution will asks for retriving mail configuration, choose "Local", evolution will asks for file with mails, run previous script and file will be created, and placed at /var/mail/mac

**Uploadprogress**

```sh
sudo pecl install uploadprogress
```

After install create file /etc/php5/apache2/conf.d/uploadprogress.ini with following line

```ini
extension=uploadprogress.so
```

**ServerName**

Put into /etc/apache2/httpd.conf

```
ServerName local.com
```

**PHP**

Edit /etc/php5/apache2/php.ini

```ini
error_reporting = E_ALL & ~E_NOTICE
display_errors = On
display_startup_errors = On
track_errors = On
html_errors = On
memory_limit = 128M
post_max_size = 128M
upload_max_filesize = 128M
max_execution_time = 900
```

Example of development php.ini can be found here: `/usr/share/doc/php5-common/examples/php.ini-development`

**XDEBUG**

Edit /etc/php5/apache2/conf.d/xdebug.ini

```ini
xdebug.remote_enable=On
xdebug.remote_enable=1
xdebug.remote_handler=dbgp
xdebug.remote_mode=req
xdebug.remote_host=127.0.0.1
xdebug.remote_port=9000
```

**Permissions**

```sh
sudo chgrp -R www-data /var/www
sudo chmod -R 777 /var/www
sudo chmod -R g+st /var/www
sudo usermod -a -G www-data mac #YOUR USERNAME HERE
```

**Sun Java**

```sh
sudo add-apt-repository ppa:ferramroberto/java
sudo apt-get update
sudo apt-get install sun-java6-jdk sun-java6-plugin
sudo update-alternatives --config java
```

**Rewrite**

Add to /etc/php5/apache2/php.ini

```ini
auto_prepend_file = /var/www/vhosts/virtual.prepend.php
```

with following content:

```php
<?php
$http_host = explode('.',$_SERVER['HTTP_HOST']);
$__mod_vhost_alias_fix_doc_root = dirname(__FILE__) . DIRECTORY_SEPARATOR . $http_host[0];

if (is_dir($__mod_vhost_alias_fix_doc_root))
{
$_SERVER['__MOD_VHOST_FIX_OLD_DOCUMENT_ROOT'] = $_SERVER['DOCUMENT_ROOT'];
$_SERVER['DOCUMENT_ROOT'] = $__mod_vhost_alias_fix_doc_root;
}
?>
```

**Links**

http://www.vmirgorod.name/11/1/20/drupal-development-environment-based-ubuntu-1010
