---
layout: post
title: Autodiscover Selenium Hub with UDP broadcast messages
tags: [selenium, grid, udp, broadcast, autodiscover]
---

Here is how we implement autodiscover of selenium hub with udp broad cast messages.

We are starting not only selenium hub on master machine but also small echo udp server, wich listening for udp messages on 4444 port.

Now before starting selenium nodes all we need to do is to send broadcast message on 4444 port and catch IP from master answer message, after that we know hub ip and can easily register our new node.

In my case, server is running under windows, here is files:

**Server.cmd**

```sh
@ECHO OFF
START powershell -ExecutionPolicy Bypass -File "%~dp0Scripts\UdpServer.ps1"
"%~dp0Portables\jre\bin\java.exe" -jar "%~dp0Portables\selenium-server-standalone.jar" -role hub
```

**Scripts\UdpServer.ps1**

```powershell
$IPAddress = Get-NetIPAddress -AddressFamily IPv4 -AddressState Preferred | ?{ -not ($_.IPAddress -eq '127.0.0.1') } | select -First 1 IPAddress -ExpandProperty IPAddress
$Port=4444

$Client = New-Object System.Net.Sockets.UdpClient $Port
$ListenEndpoint = New-Object System.Net.IPEndPoint([IPAddress]::Any, $Port)

Write-Host "$($IPAddress):$($Port) - LISTENING" -ForegroundColor Green

while($True) {
    $Client.Receive([ref]$ListenEndpoint)
    $IP = $ListenEndpoint.Address.ToString()

    if(-not($IPAddress -eq $IP)) {

        <#
        Start-Sleep -Milliseconds 100
        $Temp = New-Object System.Net.Sockets.UdpClient
        $ResponseEndpoint = New-Object System.Net.IPEndPoint ([IPAddress]::Parse($IP), $Port)
        $BytesSent = $Temp.Send(@(), 0, $ResponseEndpoint)
        $Temp.Close()
        #>

        $job = Start-Job -ScriptBlock {
            Start-Sleep -Milliseconds 100
            $Temp = New-Object System.Net.Sockets.UdpClient
            $ResponseEndpoint = New-Object System.Net.IPEndPoint ([IPAddress]::Parse($args[0]), $args[1])
            $BytesSent = $Temp.Send(@(), 0, $ResponseEndpoint)
            $Temp.Close()
        } -ArgumentList $IP, $Port

        Write-Host $IP

    }
}
```

And here is worker files:

**Worker.cmd**

```sh
@ECHO OFF
FOR /F %%i IN ('powershell -ExecutionPolicy Bypass -File "%~dp0Scripts\HubDiscover.ps1"') DO SET SELENIUM_HUB_IP=%%i
"%~dp0Portables\jre\bin\java" -jar "%~dp0Portables\selenium-server-standalone.jar" -role node -hub http://%SELENIUM_HUB_IP%:4444/grid/register -browser "browserName=firefox, firefox_binary=%~dp0Portables\Firefox\firefox.exe, maxInstances=2"
```

**Scripts\HubDiscover.ps1**

```powershell
$Port = 4444

$EndPoint = New-Object System.Net.IPEndPoint ([IPAddress]::Broadcast, $Port)
$Client = New-Object System.Net.Sockets.UdpClient
$BytesSent = $Client.Send(@(), 0,$EndPoint)
$Client.Close()

$EndPoint = New-Object System.Net.IPEndPoint ([IPAddress]::Any, $Port)
$Client = New-Object System.Net.Sockets.UdpClient $Port
$Content = $Client.Receive([ref]$EndPoint)
$IP = $EndPoint.Address.ToString()
$Client.Close()

Write-Host $IP
```

Now you can run Selenium Grid from any machine in your local network and all nodes started after will automatically connect to it.

There is SSDP (Simple Service Discovery Protocol) built into Windows, wich works in same way, so probably udp messages can be somehow adopted for it.

Also think of udp servers on nodes, that can accept command messages and restart nodes - it will be awesome addition.

Here is linux part:

Vagrant file creating multiple machines

**Vagrantfile**

```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "hashicorp/precise64"
    config.vm.provision "shell", path: "Scripts/Provision.sh"
    config.vm.provision "shell", path: "Scripts/Worker.sh", run: "always"

    1..4.times do |index|
    config.vm.define "node#{index}" do |node|
    end
    end
end
```

You should add public network to vm definition if you are using virtualbox, but for hyper-v there is no need to do it.

**Scripts/Provision.sh**

```sh
#!/usr/bin/env bash

# Force Ubuntu use closest mirrors
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise-updates main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise-backports main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list
echo "deb mirror://mirrors.ubuntu.com/mirrors.txt precise-security main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list

# Install required software
sudo apt-get update
sudo apt-get install -y openjdk-7-jre firefox xvfb x-ttcidfont-conf xfonts-100dpi xfonts-75dpi xfonts-scalable xfonts-cyrillic xserver-xorg-core
wget "http://selenium-release.storage.googleapis.com/2.42/selenium-server-standalone-2.42.2.jar"
sudo mv selenium-server-standalone-2.42.2.jar /usr/local/bin/selenium-server-standalone.jar
```

**Scripts/Worker.sh**

```sh
#!/usr/bin/env bash

export DISPLAY=:0
Xvfb :0 -screen 0 1024x768x24 -ac 2>&1 >/dev/null &
SELENIUM_HUB_IP=`python /vagrant/Scripts/HubDiscover.py`
nohup java -jar /usr/local/bin/selenium-server-standalone.jar -role node -hub http://$SELENIUM_HUB_IP:4444/grid/register -browser "browserName=firefox, maxInstances=2" &
```

**Scripts/HubDiscover.py**

```python
#!/usr/bin/env python

import socket
import sys
import os


# Send broadcast message
client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
client.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
client.sendto('', ('<broadcast>' , 4444))
client.close()


# Catch response
client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
client.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
client.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
client.bind(('', 4444))
message , address = client.recvfrom(4444)
client.close()


print address[0]
```
