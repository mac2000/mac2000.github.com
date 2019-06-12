---
layout: post
title: docker localhost ssl
tags: [docker, docker-compose, mkcert, ssl, self-signed]
---

Two ways of having trusted self signed certs for local development needs

First one with [mkcert](https://github.com/FiloSottile/mkcert) which will add root ca to system so it will trust it

Installation and generating certificates is as easy as:

```bash
brew install mkcert
mkcert -install
mkcert -cert-file vcap.me.crt -key-file vcap.me.key "*.vcap.me"
cp "$(mkcert -CAROOT)/rootCA.pem" ca.crt
```

Note that I'm using `vcap.me` which is resolving to `127.0.0.1`

And here is `docker-compose.yml`

```yml
version: "3.5"
services:
  nginx-proxy:
    container_name: proxy
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - ./certs:/etc/nginx/certs
    networks:
      default:
        ipv4_address: 172.20.0.200

  whoami:
    container_name: whoami
    image: jwilder/whoami
    volumes:
      - ./certs/ca.crt:/usr/local/share/ca-certificates/ca.crt
    environment:
      - VIRTUAL_HOST=whoami.vcap.me
      - VIRTUAL_PORT=8000
    extra_hosts:
      - "nginx.vcap.me:172.20.0.200"
      - "whoami.vcap.me:172.20.0.200"

  nginx:
    container_name: nginx
    image: nginx:alpine
    volumes:
      - ./certs/ca.crt:/usr/local/share/ca-certificates/ca.crt
    environment:
      - VIRTUAL_HOST=nginx.vcap.me
      - VIRTUAL_PORT=80
    extra_hosts:
      - "nginx.vcap.me:172.20.0.200"
      - "whoami.vcap.me:172.20.0.200"

networks:
  default:
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

With such setup ssl will work not only from outside docker but between containers alsto.

There is [localhost.tools](https://localhost.tools/) guys registered domain and configured lets encrypt wild card certificates which allows to achieve the same result without installing anything on system like this:

```yml
version: "3.5"
services:
  proxy:
    container_name: proxy
    image: tarampampam/localhost
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      default:
        ipv4_address: 172.20.0.200

  whoami:
    container_name: whoami
    image: jwilder/whoami
    labels:
      traefik.frontend.rule: Host:whoami.localhost.tools
      traefik.protocol: http
      traefik.port: 8000
    extra_hosts:
      - "whoami.localhost.tools:172.20.0.200"
      - "api.localhost.tools:172.20.0.200"

  api:
    container_name: api
    build:
      context: api
      dockerfile: Dockerfile
    volumes:
      - ./api:/app
      - /api/bin
      - /api/obj
    labels:
      traefik.frontend.rule: Host:api.localhost.tools
      traefik.protocol: http
      traefik.port: 5000
    extra_hosts:
      - "whoami.localhost.tools:172.20.0.200"
      - "api.localhost.tools:172.20.0.200"

networks:
  default:
    ipam:
      config:
        - subnet: 172.20.0.0/16
```
