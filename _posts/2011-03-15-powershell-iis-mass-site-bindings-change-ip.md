---
layout: post
title: PowerShell IIS mass site bindings change ip

tags: [admin, batch, binding, iis, ip, mass, network, powershell]
---

If u do not do so before run:

```powershell
Set-ExecutionPolicy remotesigned
```

Not right click on PowerShell and choose Import all modules.

Or in powershell try this:

```powershell
Import-Module WebAdministration
Get-Command WebAdministration\*
```

Here is some samples that was on my way to dzen

Setting up bindings examples:

```powershell
Set-WebConfigurationProperty "/system.applicationHost/sites/site[@name='local.rabota.ua']/bindings/binding[@protocol='http']" -name bindingInformation -value '127.0.0.1:80:local.rabota.ua'
Set-WebConfigurationProperty "/system.applicationHost/sites/site[@name='local.rabota.ua']/bindings/binding[@protocol='http']" -name bindingInformation -value '[::1]:80:local.rabota.ua'
```

List domain <-> site

```powershell
$sites = @{}
Get-Website | foreach { $site_name = $_.name; Get-WebBinding -name $site_name | foreach { $sites.Add([regex]::replace($_.bindingInformation, '.*?:80:', ''), $site_name) } }
$sites
```

And here it is:

```powershell
Get-WebConfigurationProperty -filter "/system.applicationHost/sites/site/bindings/binding[@protocol='http']" -name bindingInformation | foreach { $site_binding = [regex]::replace($_.itemXPath, ".*?@bindingInformation='.*?:80:(.*?)'.*", "`$1"); Set-WebConfigurationProperty $_.itemXPath -name bindingInformation -value "[::1]:80:$site_binding" }
```

We are getting all bindings, then from xpath we getting domain name, and change binding to new ip.

Now u can make sheduled task on dhcp ip change to change all sites ip

And here is sample how i change ip

```powershell
net stop w3svc
netsh http show iplisten
netsh http delete iplisten ::1
netsh http add iplisten ipaddress="192.168.56.1:80"
netsh http show iplisten
Get-WebBinding
Get-WebConfigurationProperty -filter "/system.applicationHost/sites/site/bindings/binding[@protocol='http']" -name bindingInformation | foreach { $site_binding = [regex]::replace($_.itemXPath, ".*?@bindingInformation='.*?:80:(.*?)'.*", "`$1"); Set-WebConfigurationProperty $_.itemXPath -name bindingInformation -value "192.168.56.1:80:$site_binding" }
Get-WebBinding
net start w3svc
```

Do not forget to change your hosts file, but better setup DNS

When run power shell do not forget to run:

```powershell
import-module webadministration
```

After some research noticed that there is no need to change webbinding, so script going to be more easier:

```powershell
$connection_name = "lan"
Import-Module WebAdministration
Get-Website | foreach { Stop-Website $_.name }
net stop w3svc
$connection_index = (gwmi -query "SELECT Index FROM Win32_NetworkAdapter WHERE NetConnectionID='$connection_name'").Index
$new_ip = (gwmi -query "SELECT IPAddress FROM Win32_NetworkAdapterConfiguration WHERE Index=$connection_index").IPAddress | select-string -Pattern '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'
netsh http show iplisten
$current_iplisten_ips = netsh http show iplisten | select-string -Pattern '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' -AllMatches | foreach { $_.Matches } | foreach { $_.Value }
$current_iplisten_ips | foreach { netsh http delete iplisten $_ }
netsh http add iplisten ipaddress=$new_ip
netsh http show iplisten
net start w3svc
Get-Website | foreach { Start-Website $_.name }
```

All we do is change iplisten, and in IIS leave all binding listen `*`.
