---
layout: page
title: Linux console
---

Change multiple files extension
-------------------------------

    rename 's/\.css$/\.scss/' *.css

Backup(restore) directory
-------------------------

    # backup
    tar -cpzf test-`date +%Y-%m-%d`.tar.gz /home/mac/test
    # restore
    tar -xpzf /home/mac/test-2012-04-25.tar.gz -C /

Backup(restore) mysql
---------------------

    # backup
    mysqldump --databases -u [LOGIN] --password=[PASSOWRD] [DATABASE] > /home/mac/db-`date +%Y-%m-%d`.sql
    # restore
    mysql -u [LOGIN] --password=[PASSWORD] [DATABASE] < /home/mac/db-2012-04-25.sql

If there is errors about importing big file, set `set global max_allowed_packet=128*1024*1024;`

Symlink
-------

    ln -s [existing_file] [link]
    ln -s /var/www /home/mac/www

Search for text in files
------------------------

    grep -H -r "TODO" | cut -d: -f1

Sync with production server
---------------------------

    rsync -avz -e ssh [USERNAME]@[HOST]:/var/www/ ./
    mysqldump --add-drop-database --databases -h [HOST] -u [USERNAME] --password=[PASSWORD] [DATABASE] > [FILE_NAME].sql
    mysql -u root --password=[PASSWORD] < [FILE_NAME].sql

First command will sync files, two next - database

Remove line from multiple files
-------------------------------

    find . -name "*.md" -exec sed -i 's/^permalink.*$//' {} \;

Replace line in a file
----------------------

    sed -i 's/search_me/replacement/' test.txt

Ubuntu version
--------------

    cat /etc/issue

Upgrade ubuntu
--------------

    apt-get install update-manager-core
    do-release-upgrade

Ubuntu locales
--------------

    apt-get install language-pack-ru
    update-locale LC_ALL=ru_UA.UTF-8 LANG=ru_UA.UTF-8

Notice that I used `ru_UA` for Ukrainian instead of `ru_RU`
To get effect logout from system and login back

To regenerate locale use `sudo locale-gen en_US.UTF-8`

Allow user use sudo (add him to sudo group)
-------------------------------------------

    #usermod -aG <groupname> <username>
    usermod -aG sudo <username>

Automatically login
-------------------

Replace last line in `/etc/init/tty1.conf` from `exec /sbin/getty -8 38400 tty1` to `exec /bin/login -f USERNAME < /dev/tty1 > /dev/tty1 2>&1`

Show available network cards
----------------------------

    ifconfig | grep "^[a-z]" | cut -d":" -f 1

SSH authorize with keys
-----------------------

On your laptop run:

    ssh-keygen

Answer with default values to all questions. Then run:

    ssh-copy-id user@server

Change `user` and `server` to apropriate values. This will allow you to connect to remote machine without entering password.

On Windows, run `puttygen.exe`, generate `SSH-2 RSA` with 1024 bits and save generated keys (private.ppk and id_rsa.pub). Code to copy into `authorized_keys` will be at top of puttygen window.

Now in putty under `Connection\SSH\Auth` select generated private key, and copy your public key to servers `~/.ssh/authorized_keys`

To make it more easier, create file `~/.ssh/config` with config like this:

	Host example
		User example
		Hostname example.com

Now instead:

	ssh example@example.com

You can use:

	ssh example

For Putty, navigate to `Connection \ SSH \ Auth` and select your generated file in `Private key file for authentication`

screen
------

To start new session use: `screen [-S my]`
To reconnect to session use: `screen -r [my]`
To list available sessions use: `screen -ls`
Connect second user to screen: `screen -x [my]`


scp
---

Local to remote

    scp path/myfile user@8.8.8.8:/full/path/to/new/location/

Remote to local

    scp user@8.8.8.8:/full/path/to/file /path/to/put/here

Notice semicolon before path to remote file.
Under windows you can use `pscp.exe` wich comes with putty.

packages
--------

	dpkg --get-selections # show installed packages
	sudo dpkg -i app.dep # install package
	sudo dpkg -r app # uninstall package, run sudo apt-get autoremove after

change default editor
---------------------

	sudo update-alternatives --config editor

get top 404 urls from access.log
--------------------------------

    cat access.log | awk '{ if($9 == 404) { print $7 } }' | sort | uniq -c | sort -r | head -10

    grep ' 404 ' access.log | cut -d ' ' -f 7 |sort |uniq -c |sort -n

who tried root access via ssh
-----------------------------

cat auth.log | grep "Failed password for root from" | cut -d ' ' -f 11 | sort | uniq -c | sort -n

prepend file
------------

    sed -i -e '1i TEXT' FILE

will add string "TEXT" to FILE

memory usage per process
------------------------

ps axo user,pid,%mem,rss,cmd --sort -rss | head

# Same but cut long commands
ps axo user,pid,%mem,rss,cmd --sort -rss | head | cut -c1-$(stty size </dev/tty | cut -d' ' -f2)

Multiline echo into file
------------------------

    sudo cat >> /etc/supervisor/supervisord.conf << EOL
    [include]
    files = conf.d/*.conf
    EOL

Use `cat >>` to append to file and `cat >` to overwrite whole file

Another way for permission denied files:

    cat << EOF | sudo tee -a /etc/supervisor/supervisord.conf
    [include]
    files = conf.d/*.conf
    EOF

Use `tee -a` to append to file and `tee` to overwrite whole file

Sudo without password
---------------------

    sudo visudo

add to end of the file:

    mac ALL=(ALL) NOPASSWD: ALL

oneliner:

    echo administrator ALL=NOPASSWD: ALL | sudo tee /etc/sudoers.d/administrator

sed remove comments
-------------------

multiline commment blocks

	sed '/\/\*/,/\*\//d inputFile

single line comments

	sed 's/#.*$//g' FileName
