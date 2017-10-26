---
layout: post
title: Linux iptables sample for lamp

tags: [centos, filter, firewall, iptables, linux, netstat, ubuntu]
---

```sh
#!/bin/sh

#save this to firewall.sh, and make chmod +x firewall.sh
#available ports: icmp, tcp, udp
#to see what is running on what port: netstat -tulpn

#flush all rules
/sbin/iptables -F

#allow all trafic on localhost interface
/sbin/iptables -A INPUT -i lo -j ACCEPT

#allow icmp (ping) packets
/sbin/iptables -A INPUT -p icmp -j ACCEPT

#allow input trafic that was initiated from us
/sbin/iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#allow input trafic for specific portsz
/sbin/iptables -A INPUT -p tcp --dport 20 -j ACCEPT
/sbin/iptables -A INPUT -p tcp --dport 21 -j ACCEPT
/sbin/iptables -A INPUT -p tcp --dport 22 -j ACCEPT
/sbin/iptables -A INPUT -p tcp --dport 80 -j ACCEPT
/sbin/iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
/sbin/iptables -A INPUT -p tcp --dport 1024:65535 -j ACCEPT

#default action for other packets
/sbin/iptables -P INPUT DROP
/sbin/iptables -P FORWARD DROP
/sbin/iptables -P OUTPUT ACCEPT

#save rules
/sbin/service iptables save

#list rules
/sbin/iptables -L -v
```


Another variant is to:

```sh
apt-get install iptables-persistent
```

Now run all commands above to add firewall rules, and run:

```sh
iptables-save > /etc/iptables/rules
```

If you will add new rule laterm do not forget to add them using command above
