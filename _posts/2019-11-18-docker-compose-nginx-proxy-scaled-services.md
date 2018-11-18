---
layout: post
title: Nginx Proxy saceld docker-compose services
tags: [docker-compose, nginx-proxy, scale]
---

Easiest way to proxy scaled containers

**docker-compose.yml**

```yml
version: "3"

services:
  proxy:
    image: jwilder/nginx-proxy:alpine
    container_name: proxy
    ports:
      - 80:80
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
    logging:
      driver: none

  whoami:
    image: jwilder/whoami
    ports:
      - 8000
    environment:
      VIRTUAL_HOST: localhost
```

Now if you run:

```bash
docker-compose up
docker-compose scale whoami=5
```

And open localhost in browser, each time you refresh the page you will see different container, and you can still access concrete containers by their own ports

```bash
$ curl http://localhost
I'm a9998f9cefbd

$ curl http://localhost
I'm 66d5e9d72b61

$ curl http://localhost
I'm 6bab17a8d614

$ curl http://localhost
I'm 103bac359b56

$ docker ps --format "{{.ID}}: {{.Ports}}" --filter ancestor=jwilder/whoami
a9998f9cefbd: 0.0.0.0:32791->8000/tcp
66d5e9d72b61: 0.0.0.0:32790->8000/tcp
6bab17a8d614: 0.0.0.0:32789->8000/tcp
103bac359b56: 0.0.0.0:32788->8000/tcp
de52272cac61: 0.0.0.0:32787->8000/tcp

$ curl http://localhost:32790
I'm 66d5e9d72b61
```
