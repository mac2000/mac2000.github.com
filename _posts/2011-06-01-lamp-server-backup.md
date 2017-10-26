---
layout: post
title: Lamp server backup

tags: [automysqlbackup, backup, centos, cron, link, linux, ln, simplebashbu, ubuntu, wget]
---

Automatic creation of archived daily, weekly, monthly backups of sites and databases  with email notification.

Found at: http://www.nardin.info/2010/07/backup-lamp.html

Scripts:

http://sourceforge.net/projects/automysqlbackup/

http://sourceforge.net/projects/simplebashbu/

put them somewhere and make them executable.

```sh
#make skripts executable
chmod +x /home/simplebashbu.sh
chmod +x /home/automysqlbackup.sh
#make links of scripts to run every day
ln -s /home/simplebashbu.sh /etc/cron.daily/
ln -s /home/automysqlbackup.sh /etc/cron.daily/
```

In my centos mail was not installed so simlebashby was not able to send email, to fix run:

```sh
yum install mailx
```

To download backups:

```sh
wget -m ftp://LOGIN:PASSWORD@example.com/backups -P C:\Users\mac\Desktop\
```

Wget for windows: http://gnuwin32.sourceforge.net/packages/wget.htm

Download binaries and dependencies, extract all to same directory.
