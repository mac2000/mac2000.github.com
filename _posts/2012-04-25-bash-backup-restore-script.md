---
layout: post
title: Bash backup-restore script

tags: [backup, bash, cat, chmod, drush, gz, linux, ls, menu, mysql, mysqldump, restore, script, sed, sh, tail, tar, touch, ubuntu, var, variable, whiptail]
---

Backup\Restore Commands
-----------------------

**Backup files**

```sh
/bin/tar -cpzf /path-to-backups/www-`date +%Y-%m-%d`.tar.gz /home/example/www/
```

**Restore files**

```sh
/bin/tar -xpzf /path-to-backups/www-2012-04-25.tar.gz -C /
```

**Backup mysql**

```sh
/usr/bin/mysqldump --databases -u MYSQL_LOGIN --password=MYSQL_PASSWORD MYSQL_DATABASE > /path-to-backups/db-`date +%Y-%m-%d`.sql
```

**Restore mysql**

```sh
/usr/bin/mysql -u MYSQL_LOGIN --password=MYSQL_PASSWORD MYSQL_DATABASE < /path-to-backups/db-2012-04-25.sql
```

http://help.ubuntu.com/community/BackupYourSystem/TAR

Backup Script
-------------

```sh
touch ~/make-backup.sh
chmod a+x ~/make-backup.sh
sudo ln -s /home/example/make-backup.sh /etc/cron.daily/make-backup.sh
cat ~/make-backup.sh
#!/bin/sh

# Backup www directory
/bin/tar -cpzf /home/example/backups/www-`date +%Y-%m-%d`.tar.gz /home/example/www/

# Backup database
/usr/bin/mysqldump --databases -u example --password=example example > /home/example/backups/db-`date +%Y-%m-%d`.sql

# Delete old backups
/usr/bin/find /home/example/backups/ -mtime +30 -delete
```

Restore Script
--------------

```sh
touch ~/restore-backup.sh
chmod a+x ~/restore-backup.sh
cat ~/restore-backup.sh
#!/bin/sh

# Get last backup file names into variables
LAST_WWW_BACKUP=$(ls /home/example/backups/www-*.tar.gz | tail -n 1)
LAST_SQL_BACKUP=$(ls /home/example/backups/db-*.sql | tail -n 1)

# Restore files
/bin/tar -xpzf $LAST_WWW_BACKUP -C /

# Restore database
/usr/bin/mysql -u example --password=example example < $LAST_SQL_BACKUP

# Clear drupal caches
drush -r /home/example/www/ cc all
```

Interactive Restore Script
--------------------------

<amp-img src="/images/wp/132.png" alt="screenshot" width="670" height="420"></amp-img>

```sh
touch ~/date-backup-restore.sh
chmod a+x ~/date-backup-restore.sh
cat ~/date-backup-restore.sh

#!/bin/sh

# Disaplay choose dialog with available backups
CHOSEN_DATE=$(whiptail --title "RESTORE BACKUP" --menu "Chose Backup Date" 20 78 10 `for x in /home/example/backups/*.tar.gz; do echo "$x backup" | sed 's/.*www-\(.*\).tar.gz/\1/'; done` 3>&1 1>&2 2>&3)

exitstatus=$?
if [ $exitstatus = 0 ]; then
        # Get files and database backup file names
        WWW_BACKUP=$(ls /home/example/backups/www-"$CHOSEN_DATE".tar.gz | tail -n 1)
        SQL_BACKUP=$(ls /home/example/backups/unon-$CHOSEN_DATE.sql | tail -n 1)

        # Restore files
        /bin/tar -xpzf $WWW_BACKUP -C /

        # Restore database
        /usr/bin/mysql -u example --password=example example < $SQL_BACKUP

        # Clear drupal caches
        drush -r /home/example/www/ cc all
fi
```

TODO: refactor scripts to use path variables

TODO: move to github
