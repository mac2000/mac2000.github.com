---
layout: post
title: PowerShell progress bar

tags: [automate, cmd, powershell, shell, write-progress, activity, percentcomplete, currentoperation, innerloop, start-sleep]
---

```powershell
for($i = 1; $i -le 100; $i++) {write-progress -activity "Search in Progress" -status "$i% Complete:" -percentcomplete $i;Start-Sleep 1;}
for($j = 1; $j -lt 101; $j++) {write-progress -id  1 -activity Updating -status 'Progress' -percentcomplete $j -currentOperation InnerLoop}
```
