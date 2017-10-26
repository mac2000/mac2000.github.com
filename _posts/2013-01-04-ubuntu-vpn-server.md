---
layout: post
title: Ubuntu as VPN server
tags: [pptp, pptpd, ubuntu, vpn]
---

```sh
sudo apt-get install pptpd
```

Edit `/etc/pptpd.conf`

```
bcrelay venet0:0
localip 192.168.78.1
remoteip 192.168.78.100-110
```

Edit `/etc/ppp/chap-secrets`

```
mac pptpd 123 * # login, server, password, ip
```

Edit `/etc/rc.local`

```
# PPTP IP forwarding
iptables -t nat -A POSTROUTING -o venet0:0 -j MASQUERADE
exit 0
```

You can run this command without reboot.

Edit `/etc/sysctl.conf`

```
net.ipv4.ip_forward=1
```

To write value without reboot use:

```
sudo sysctl -w net.ipv4.ip_forward=1
```
