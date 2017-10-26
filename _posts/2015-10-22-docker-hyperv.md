---
layout: post
title: Docker on Windows Hyper-V
tags: [docker, hyperv, provision, tls, certificates]
---

<iframe width="420" height="315" src="https://www.youtube.com/embed/eQ40Tx0F8Yc" frameborder="0" allowfullscreen></iframe>

Provision of docker host
------------------------

I'm gonna use Ubuntu 15.04, at moment there is troubles with `DOCKER_OPTS` hope will be fixed later on.

On a fresh install all you need to do:

###Install docker itself

```sh
wget -qO- https://get.docker.com/ | sh
sudo usermod -aG docker $USER # relogin required
docker run hello-world
```

###Certificates

https://docs.docker.com/articles/https/

How it works: Both client and server have its own cert and key files, they all are signed with same CA. While connecting both client and server verify each other certificates agains CA file they have. If all OK communication begins.

**First of all we need our CA**

```sh
openssl genrsa -aes256 -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -subj "/CN=*" -out ca.pem
```

**Server certificates**

```sh
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=*" -sha256 -new -key server-key.pem -out server.csr
echo subjectAltName = IP:192.168.5.123,IP:127.0.0.1 > extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
rm server.csr extfile.cnf ca.srl
```

Here is tricky part, you need to provide your server IP here to be able to connect to it by IP address rather than by DNS.

But in my case its my local lab so I prefer doing it without IP addresses like so:

```sh
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=*" -sha256 -new -key server-key.pem -out server.csr
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem
rm server.csr ca.srl
```

Not that in this variant I am no longer defining IP addresses so connections will be available only by host names which in our case everywhere set to start - so connection will be available from everywhere, but from now on you must deal with hosts file on your client system or configure DNS.

**Client certificates**

```sh
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
rm client.csr extfile.cnf ca.srl
```

**Privileges**

```sh
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem
```

What do we have now is:

* `server-key.pem`, `server-cert.pem` and `ca.pem` - will be used by docker server
* `key.pem`, `cert.pem` and `ca.pem` - will be used by client

Notice that both client and server do need to have same `ca.pem` to be able to verify certificates.

###DOCKER_OPTS

By default docker listens to socket, what wee need is to modify docker startup parameters so it will start listening network.

```sh
sudo sed -i "s|#DOCKER_OPTS=\"--dns 8.8.8.8 --dns 8.8.4.4\"|DOCKER_OPTS=\"--tlsverify --tlscacert=$PWD/ca.pem --tlscert=$PWD/server-cert.pem --tlskey=$PWD/server-key.pem -H=0.0.0.0:2376\"|" /etc/default/docker
```

Be sure to set appropriate file names and paths, do not forget to remove `-i` parameter to check what will happen.

Unfortunatelly at this moment things will not happen, because of moving to systemd file we were edit is not used at all, hope it will be fixed soon, but for now we need do few more things.

```sh
sudo mkdir -p /etc/systemd/system/docker.service.d

cat << EOF | sudo tee -a /etc/systemd/system/docker.service.d/ubuntu.conf
[Service]
EnvironmentFile=/etc/default/docker
ExecStart=
ExecStart=/usr/bin/docker -d -H fd:// \$DOCKER_OPTS
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status docker
netstat -ntulp | grep 2376
```

Windows client
--------------

Windows client may be downloaded here: https://get.docker.com/builds/Windows/x86_64/docker-latest.exe

To copy certificate files use something like this:

```sh
scp alexandr@192.168.5.123:"/home/alexandr/ca.pem /home/alexandr/cert.pem /home/alexandr/key.pem" .
```

To verify things are working:

```sh
docker --tlsverify --tlscacert=ca.pem --tlscert=cert.pem --tlskey=key.pem -H=hvd:2376 version
```

Note that we are connection to `hvd` machine rather that its IP.

To make things easier you may set environment variables:

```sh
set DOCKER_TLS_VERIFY=1
set DOCKER_HOST=tcp://hvd:2376
set DOCKER_CERT_PATH=C:\Users\Alexandr\Desktop
```

And now you should be able to run docker commands from your windows client against ubuntu vm in your hyper-v like so:

```sh
docker info | findstr "Operating System"
Operating System: Ubuntu 15.04
```

Docker Machine
--------------

Please take a note that there is [docker-machine](https://github.com/docker/machine/releases) available for Windows which supports hyper-v out of the box. All what have beed done here may be done in few commands like:

```sh
docker-machine create -d hyper-v dev
docker-machine env dev

set DOCKER_TLS_VERIFY=1
set DOCKER_HOST=tcp://192.168.5.156:2376
set DOCKER_CERT_PATH=C:\Users\Alexandr\.docker\machine\machines\dev
set DOCKER_MACHINE_NAME=dev

docker run busybox echo hello world
```

Docker machine will create docker host vm in hyper-v, will deal with certificates and so on. Even more, when you will be ready you can use docker machine to provision docker host in cloud providers without doing anything by hand.
