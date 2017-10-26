---
layout: post
title: Windows 2012 DNS Server dynamic forwarders
tags: [windows, 2012, dns, dhcp, forwarders]
---

I do not found solution, so will try to do it via scheduled task.

Here is what we will do:

On network connection - retrieve recieved by DHCP IP addresses of DNS servers and set them as our local DNS Server forwarders.

Then we need to swich our network adapter to use our local DNS instead of recieved one.

On network disconnect - we neeed to switch our network adapter back to accept DNS settings by DHCP.

*In all following examples wifi0 - is my network adapter*

Retrieve obtained DNS servers
-----------------------------

```powershell
(Get-DnsClientServerAddress -InterfaceAlias "wifi0" -AddressFamily IPv4).ServerAddresses
```


Set DNS Server forwarders
-------------------------

```powershell
Set-DnsServerForwarder -IPAddress (Get-DnsClientServerAddress -InterfaceAlias "wifi0" -AddressFamily IPv4).ServerAddresses -PassThru
```

Set network adapter to use static DNS
-------------------------------------

```powershell
Set-DnsClientServerAddress -InterfaceAlias "wifi0" -ServerAddresses ("127.0.0.1","8.8.8.8")
```

Set network adapter to use dynamic DNS
--------------------------------------

```powershell
Set-DnsClientServerAddress –InterfaceAlias "wifi0" -ResetServerAddresses
```

Events
------

Look in **Event Viewer** for events under

```
Event Viewer \ Applications and Services Logs \ Microsoft \ Windows \ NetworkProfile \ Operational
```

There is two kind of events with IDs 10000 for connect and 10001 for disconnect so we can use them for scheduled tasks.

```
Event Viewer \ Applications and Services Logs \ Microsoft \ Windows \ WLAN-AutoConfig \ Operational
```

There is also two events, 11010 - for connection and 11004 - for disconnection.

Now you can create two scheduled tasks.

On network connection - retrieve DNS servers and set them as forwarders for local server, then switch network adapter to use local DNS server.

On network disconnection - set network adapter back to use DHCP.

connect.ps1
-----------

```powershell
$NetworkAdapterName = "wifi0"

# Get network adapter DNS servers and set them as local DNS Server forwarders
Set-DnsServerForwarder -IPAddress (Get-DnsClientServerAddress -InterfaceAlias $NetworkAdapterName -AddressFamily IPv4).ServerAddresses

# Set network adapter DNS server to use local DNS Server
Set-DnsClientServerAddress -InterfaceAlias $NetworkAdapterName -ServerAddresses "127.0.0.1"
```

disconnect.ps1
--------------

```powershell
$NetworkAdapterName = "wifi0"

# Set Google DNS as forwarder
Set-DnsServerForwarder -IPAddress "8.8.8.8"

# Set network adapter to use DHCP to retrieve DNS settings
Set-DnsClientServerAddress –InterfaceAlias $NetworkAdapterName -ResetServerAddresses
```

Links
-----

http://www.powershellpro.com/powershell-tutorial-introduction/powershell-wmi-methods/
http://superuser.com/questions/262799/how-to-launch-a-command-on-network-connection-disconnection
http://social.technet.microsoft.com/Forums/en-US/winserverpowershell/thread/2aef17a1-6afa-43ef-a6de-17562c19833b/
http://www.thomasmaurer.ch/2012/04/replace-netsh-with-windows-powershell-basic-network-cmdlets/
