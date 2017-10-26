---
layout: post
title: HAproxy provision sample
tags: [haproxy, provision, vagrant, vagrantfile]
---

Here is simple example of HAproxy

**Vagrantfile**

```ruby
Vagrant.configure("2") do |config|
    config.vm.define "web1" do |node|
    node.vm.hostname = "web1"
    node.vm.box = "wildetech/hyper-u1404"
    node.vm.provision :shell, path: "Web.sh"
    end

    config.vm.define "web2" do |node|
    node.vm.hostname = "web2"
    node.vm.box = "wildetech/hyper-u1404"
    node.vm.provision :shell, path: "Web.sh"
    end

    config.vm.define "ha" do |node|
    node.vm.hostname = "ha"
    node.vm.box = "wildetech/hyper-u1404"
    node.vm.provision :shell, path: "HA.sh"
    end
end
```

**Web.sh**

```sh
#!/usr/bin/env bash

sudo apt-get update
sudo apt-get install -y apache2 php5 libapache2-mod-php5

sudo rm /var/www/html/index.html
sudo ln -s /vagrant/index.php /var/www/html/index.php
```

**HA.sh**

```sh
#!/usr/bin/env bash

sudo apt-get update
sudo apt-get install -y haproxy

sudo sed -i 's/ENABLED=0/ENABLED=1/' /etc/default/haproxy

WEB1_IP=$(host web1 | awk '/has address/ { print $4 ; exit }')
WEB2_IP=$(host web2 | awk '/has address/ { print $4 ; exit }')

sudo tee --append /etc/haproxy/haproxy.cfg <<EOF

frontend www
        bind *:80
        mode http
        default_backend nodes

backend nodes
        mode http
        balance roundrobin
        option forwardfor
        server web1 $WEB1_IP:80 check
        server web2 $WEB2_IP:80 check

listen stats *:1936
        stats enable
        stats uri /
        stats auth vagrant:vagrant
EOF

sudo service haproxy restart
```

**index.php**

```php
<?php sleep(1); ?>
<h1 style="text-align:center">
<?php echo `uname -n` ?>
<?php echo date('h:i:s') ?>
</h1>
```

Now you will be able to access: http://ha/ which will be balanced with roundrobin between two nodes (take a note that by making one request from browser you probably will always get to same node, use something like `ab`)

http://ha:1936/ - haproxy stats

And yet another example for memcached servers:

**Vagrantfile**

```ruby
Vagrant.configure("2") do |config|
    config.vm.define "memcached1" do |node|
    node.vm.hostname = "memcached1"
    node.vm.box = "wildetech/hyper-u1404"
    node.vm.provision :shell, path: "memcached.sh"
    end

    config.vm.define "memcached2" do |node|
    node.vm.hostname = "memcached2"
    node.vm.box = "wildetech/hyper-u1404"
    node.vm.provision :shell, path: "memcached.sh"
    end

    config.vm.define "ha" do |node|
    node.vm.hostname = "ha"
    node.vm.box = "wildetech/hyper-u1404"
    node.vm.provision :shell, path: "ha.sh"
    end
end
```

**memcached.sh**

```sh
#!/usr/bin/env bash

sudo apt-get update
sudo apt-get install memcached

sudo sed -i 's/-l 127.0.0.1/-l 0.0.0.0/' /etc/memcached.conf
sudo service memcached restart
```

**ha.sh**

```sh
#!/usr/bin/env bash

sudo apt-get update
sudo apt-get install -y haproxy

sudo sed -i 's/ENABLED=0/ENABLED=1/' /etc/default/haproxy

sudo sed -i 's/mode\(\s*\)http/mode\1tcp/' /etc/haproxy/haproxy.cfg
sudo sed -i 's/option\(\s*\)httplog/option\1tcplog/' /etc/haproxy/haproxy.cfg

sudo sed -i 's/contimeout\(\s*\)5000/contimeout\12000/' /etc/haproxy/haproxy.cfg
sudo sed -i 's/clitimeout\(\s*\)50000/clitimeout\12000/' /etc/haproxy/haproxy.cfg
sudo sed -i 's/srvtimeout\(\s*\)50000/srvtimeout\12000/' /etc/haproxy/haproxy.cfg

MEMCACHED1_IP=$(host memcached1 | awk '/has address/ { print $4 ; exit }')
MEMCACHED2_IP=$(host memcached2 | awk '/has address/ { print $4 ; exit }')

sudo tee --append /etc/haproxy/haproxy.cfg <<EOF

listen memcached
    bind *:11211
    balance leastconn
    server memcached1 $MEMCACHED1_IP:11211 check
    server memcached2 $MEMCACHED2_IP:11211 check

listen stats *:1936
    mode http
    stats enable
    stats uri /
    stats auth vagrant:vagrant
EOF

sudo service haproxy restart
```


Same thing but with docker containers:

```sh
docker run --name mem1 -d -p 11212:11211 memcached
docker run --name mem2 -d -p 11213:11211 memcached

docker run -d --name ha -p 1936:1936 -p 11211:11211 --link mem1:mem1 --link mem2:mem2 -v /vagrant/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro haproxy
```

and its haproxy config:

```
global
    maxconn 4096
    #daemon #must be commented

defaults
    mode http
    timeout connect 2000ms
    timeout client 2000ms
    timeout server 2000ms

listen memcached
    bind *:11211
    mode tcp
    balance leastconn
    server mem1 ${MEM1_PORT_11211_TCP_ADDR}:${MEM1_PORT_11211_TCP_PORT} check
    server mem2 ${MEM2_PORT_11211_TCP_ADDR}:${MEM2_PORT_11211_TCP_PORT} check

listen stats *:1936
    mode http
    stats enable
    stats uri /
    stats auth vagrant:vagrant
```
