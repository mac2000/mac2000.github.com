---
layout: post
title: Docker provision
tags: [docker, provision, vagrant, windows, hyperv]
---

Here is sample of docker provision:

**Vagrantfile**

```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "wildetech/hyper-u1404"
    config.vm.hostname = "docker.vm"

    # vagrant plugin install vagrant-hostmanager
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true

    config.vm.provision :shell, path: "Provision.sh"
end
```

**Provision.sh**

```sh
#!/usr/bin/env bash

# Install latest docker
curl -sSL https://get.docker.com/ubuntu/ | sudo sh

# You can use docker without TLS like this:
# echo 'DOCKER_OPTS="-H=unix:///var/run/docker.sock -H=0.0.0.0:2375"' | sudo tee --append /etc/default/docker

# And here is "secure" way
sudo apt-get install -y git ruby-dev
git clone https://gist.github.com/sheerun/ccdeff92ea1668f3c75f certgen
sudo gem install certificate_authority
ruby certgen/certgen.rb docker.vm # You MUST generate keys for FQDN not for IP
sudo cp /root/.docker/*.pem /vagrant/
echo 'DOCKER_OPTS="--tlsverify -H=unix:///var/run/docker.sock -H=0.0.0.0:2376 --tlscacert=/root/.docker/docker.vm/ca.pem --tlscert=/root/.docker/docker.vm/cert.pem --tlskey=/root/.docker/docker.vm/key.pem"' | sudo tee --append /etc/default/docker

# Lastly allow vagrant user to do things without sudo and restart docker
sudo gpasswd -a vagrant docker
sudo service docker restart
```

Docker client for windows can be found here: https://master.dockerproject.com/

Setting environment variables for docker client:

```sh
SET DOCKER_HOST=tcp://docker.vm:2376
SET DOCKER_TLS_VERIFY=1
SET DOCKER_CERT_PATH=C:\Users\Alexandr\Desktop\Docker
```

without them you should run commands like this:

```sh
docker.exe -H tcp://192.168.137.190:2375 ps
```

if you are running without TLS, or:

```sh
docker.exe --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H tcp://docker.vm:2376 ps
```

Important note: you MUST generate keys for FQDN rather than IP, otherwise you will get "x509: cannot validate certificate for because it doesn't contain any ip sans"

Unfortunatelly at current moment `docker.exe` having troubles with color codes.

http://sheerun.net/2014/05/17/remote-access-to-docker-with-tls/ - here is manual where i have found ruby script
