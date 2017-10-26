---
layout: post
title: PowerShell run self signed scripts

tags: [admin, administration, allsigned, authenticodesignature, cert, certificate, cmd, executionpolicy, makecert, powershell, ps, ps1, shell, sign]
---

Run console as Administrator and execute following commands:

```powershell
"C:\Program Files (x86)\Microsoft SDKs\Windows\v7.0A\Bin\x64\makecert.exe" -n "CN=PowerShell Local Certificate Root" -a sha1 -eku 1.3.6.1.5.5.7.3.3 -r -sv root.pvk root.cer -ss Root -sr localMachine
"C:\Program Files (x86)\Microsoft SDKs\Windows\v7.0A\Bin\x64\makecert.exe" -pe -n "CN=PowerShell User" -ss MY -a sha1 -eku 1.3.6.1.5.5.7.3.3 -iv root.pvk -ic root.cer
```

Then from powershell with Administrator privilegies allow execute selfsigned scripts

```powershell
set-executionpolicy AllSigned
```

Then

```powershell
Set-AuthenticodeSignature C:\Users\mac\Scripts\ImagesShedule.ps1 @(Get-ChildItem cert:\CurrentUser\My -codesigning)[0]
```

makecert.exe ships with Microsoft Windows Software Development Kit, available at http://msdn.microsoft.com/en-us/windowsserver/bb980924.aspx.

Also note that makecert.exe may be located in another folder.

If all ok, running two first commands will display dialogs with promts to enter passwords.

Do not forget to run them from cmd.exe as Administrator.
