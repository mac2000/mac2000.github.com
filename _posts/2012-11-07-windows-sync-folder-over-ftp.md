---
layout: post
title: Windows sync folder over ftp
tags: [winscp, ftp, sync, robocopy, rsync]
---

On linux we have `rsync` wich can do everything. On windows there is not bad `robocopy` but it can not copy files over ftp.

Windows shares are closed for public networks and only possible way is to use VPN to connect to local network and only then you can use `robocopy` to sync files between machines.

But if, for some reasons, you can not use VPN there is another way to synchronize folder between computers via FTP.

Just install build in IIS FTP server or any other FTP server and configure it.

Download and install [WinSCP](http://winscp.net/eng/download.php). I used protable version wich contains only two executable files.

**pull.txt**

```sh
option batch abort
option confirm off
open ftp://username:password@example.com
synchronize local "%USERPROFILE%\Music" "/Music"
close
exit
```

**pull.cmd**

```sh
WinSCP.com /script=pull.txt
PAUSE
```

**push.txt**

```sh
option batch abort
option confirm off
open ftp://username:password@example.com
synchronize remote "%USERPROFILE%\Music" "/Music"
close
exit
```

**push.cmd**

```sh
WinSCP.com /script=push.txt
PAUSE
```

Both scripts will not delete any files - only synchronize new one. But if you want to delete files use `-delete` switch like this:

```sh
synchronize local -delete "%USERPROFILE%\Music" "/Music"
```

For more info about `synchronize` command go to: http://winscp.net/eng/docs/scriptcommand_synchronize

Notice here that i'm using `WinSCP.com` wich is command line client for `WinSCP.exe`.

Also notice how `local` and `remote` used to determine direction of synchronization.
