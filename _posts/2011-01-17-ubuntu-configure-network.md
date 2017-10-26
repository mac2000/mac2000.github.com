---
layout: post
title: Ubuntu configure network

tags: [admin, administration, config, dhcp, dns, eth0, ifconfig, init.d, interface, ip, network, ubuntu]
---

http://www.ubuntugeek.com/ubuntu-networking-configuration-using-command-line.html

Configs are here:

```
/etc/network/interfaces
```

DHCP

```
auto eth0
iface eth0 inet dhcp
```

Static

```
auto eth0
iface eth0 inet static
address 192.168.7.101
gateway 192.168.7.1
netmask 255.255.255.0
network 192.168.7.0
broadcast 192.168.7.255
```

Static second IP on the same interface

```
auto eth0:1
iface eth0:1 inet static
address 192.168.7.102
gateway 192.168.7.1
netmask 255.255.255.0
network 192.168.7.0
broadcast 192.168.7.255
```

Apply settings

```
sudo /etc/init.d/networking restart
```

Hostname

```
sudo /bin/hostname newname
```

DNS

```
sudo nano /etc/resolv.conf
```

in config:

```
search example.com
nameserver 192.168.7.1
```
