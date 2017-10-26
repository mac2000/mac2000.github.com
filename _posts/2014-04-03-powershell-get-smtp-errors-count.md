---
layout: post
title: PowerShell Get SMTP Errors Count
tags: [powershell, ps, smtp, eventlog]
---

Lists errors count by dates

```powershell
Get-EventLog -LogName System -Source smtpsvc | group { $_.TimeGenerated.Date } | ft @{n='Date';e={$_.Name.Split(' ')[0]}}, Count -autosize
```

Will output something like:

```
Date       Count
----       -----
03.04.2014 35349
02.04.2014 17685
```
