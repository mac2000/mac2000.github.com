---
layout: post
title: Custom Protocol Handler for Windows
tags: [protocol, schema, magnet]
---

So, we have our web CRM and desktop voip client and need to connect them together.

If you have look at `HKEY_CLASSES_ROOT\mailto` you will find example of how can you do this.

And here is sample **alert.reg**:

Windows Registry Editor Version 5.00

```ini
[HKEY_CLASSES_ROOT\alert]
@="URL:Alert Protocol"
"URL Protocol"=""

[HKEY_CLASSES_ROOT\alert\DefaultIcon]
@="C:\\Program Files\\TortoiseSVN\\bin\\TortoiseProc.exe"

[HKEY_CLASSES_ROOT\alert\shell]

[HKEY_CLASSES_ROOT\alert\shell\open]

[HKEY_CLASSES_ROOT\alert\shell\open\command]
@="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe -Command \"& {[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show($args[0])}\" \"%1\""
```

Now if you will navigate in browser to url like this: [alert://hello](alert://hello) you will see native Windows message box alert with your hello message.
