---
layout: post
title: Apt-cacher-ng provision
tags: [ubuntu, apt, cache, proxy, provision]
---

apt-cacher-ng speeds up provision of 3 lamp nodes from 12 to 4 minutes.

here is simple example how to get it up and running:


**Vagrantfile**

```ruby
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.define "server" do |node|
    node.vm.hostname = "server"
    node.vm.network "private_network", ip: "192.168.33.11"
    node.vm.provider "virtualbox" do |vb|
      vb.name = "server"
    end
    node.vm.provision :shell, path: "Server.sh"
  end

  config.vm.define "client" do |node|
    node.vm.hostname = "client"
    node.vm.network "private_network", ip: "192.168.33.12"
    node.vm.provider "virtualbox" do |vb|
      vb.name = "client"
    end
    node.vm.provision :shell, path: "Client.sh"
  end
end
```


**Server.sh**

```sh
#!/usr/bin/env bash

sudo apt-get update
sudo apt-get install -y apt-cacher-ng
```


**Client.sh**

```sh
#!/usr/bin/env bash

echo 'Acquire::http { Proxy "http://apt.mac.rabota.ua:3142"; };' | sudo tee /etc/apt/apt.conf.d/02proxy
```


**Proxy stats**

http://apt.mac.rabota.ua:3142/acng-report.html?doCount=Count+Data
