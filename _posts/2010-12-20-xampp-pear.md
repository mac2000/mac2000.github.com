---
layout: post
title: XAMPP Pear

tags: [admin, develop, pear, php]
---

**List installed packages**

```sh
C:\xampp\php\pear.bat list
```

**Install package with all dependencies**

```sh
C:\xampp\php\pear.bat install PACKAGE -a
```

**Help**

```sh
C:\xampp\php\pear.bat help COMMAND
```

**Upgrade**

```sh
C:\xampp\php\pear.bat upgrade-all
```

**Channels**

[Guide users commandline channels](http://www.developertutorials.com/pear-manual/guide.users.commandline.channels.html)

**PEAR.INI**

Add `PHP_PEAR_SYSCONF_DIR` sys variable that point to php’s dir

**BETA**

To install beta use something like this:

```sh
pear install channel://pear.php.net/PEAR_Frontend_Web-0.7.5
```

**Pear webinstaller**

```sh
pear install pear/PEAR#webinstaller
```
