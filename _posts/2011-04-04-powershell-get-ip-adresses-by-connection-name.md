---
layout: post
title: PowerShell get ip adresses by connection name

tags: [gwmi, powershell, ps, wmi]
---

```powershell
$connection_name = "lan"

$index = (gwmi -query "SELECT Index FROM Win32_NetworkAdapter WHERE NetConnectionID='$connection_name'").Index
$ipaddresses = (gwmi -query "SELECT IPAddress FROM Win32_NetworkAdapterConfiguration WHERE Index=$index").IPAddress
$ipaddresses | foreach { $_ }
```

or even better to retrive only ipv4

```powershell
$connection_name = "lan"
$index = (gwmi -query "SELECT Index FROM Win32_NetworkAdapter WHERE NetConnectionID='$connection_name'").Index
$ip = (gwmi -query "SELECT IPAddress FROM Win32_NetworkAdapterConfiguration WHERE Index=$index").IPAddress | select-string -Pattern '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'
$ip
```
