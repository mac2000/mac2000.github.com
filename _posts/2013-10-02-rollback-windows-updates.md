---
layout: post
title: Rollback Windows Updates
tags: [wusa, powershell, windows, update]
---

List installed updates:

```sh
wmic qfe list brief /format:htable > "%USERPROFILE%\Desktop\hotfix.html"
wmic qfe list brief /format:texttablewsys > "%USERPROFILE%\Desktop\hotfix.txt"
```

List installed updates (powershell):

```powershell
Get-HotFix
```

Uninstall update:

```sh
wusa.exe /uninstall /KB:2785220
wusa.exe /uninstall /KB:2785220 /quiet /norestart
```

Bach mode uninstall:

```sh
wusa.exe /uninstall /KB:2773072 /quiet /norestart & ping 1.1.1.1 -n 1 -w 5000 > nul
wusa.exe /uninstall /KB:2778344 /quiet /norestart & ping 1.1.1.1 -n 1 -w 5000 > nul
wusa.exe /uninstall /KB:2785220 /quiet /norestart & ping 1.1.1.1 -n 1 -w 5000 > nul
wusa.exe /uninstall /KB:2786081 /quiet /norestart & ping 1.1.1.1 -n 1 -w 5000 > nul
```

Bach mode uninstall (powershell):

```powershell
$Updates = ((New-Object -ComObject Microsoft.Update.Session).CreateUpdateSearcher()).Search("IsInstalled = 1").Updates # Retrieve installed updates
Write-Host $Updates.Count "updates total" -ForegroundColor Yellow

$Updates = $Updates | Where-Object {$_.LastDeploymentChangeTime -gt (Get-Date("01/01/" + ((Get-Date).Year)))} # Get updates installed in this year
#$Updates = $Updates | Where-Object {$_.LastDeploymentChangeTime -gt ((Get-Date).AddMonths(-1))} # Get updates installed in month
$Updates = $Updates | Sort-Object LastDeploymentChangeTime -Descending # Sort updates

Write-Host $Updates.Count "updates installed in selected time period" -ForegroundColor Yellow
Write-Host

#$Updates = $Updates | Select -First 2

Foreach($Update in $Updates) {
	$ID = $Update.KBArticleIDs

	Write-Host "Removing KB:$ID" -ForegroundColor Red
	Write-Host $Update.Title
	Write-Host
	Invoke-Expression "wusa.exe /uninstall /kb:$ID /quiet /log /norestart"

	# Wait wusa to complete uninstall
	while (@(Get-Process wusa -ErrorAction SilentlyContinue).Count -ne 0) { Start-Sleep 1 }
}
```
