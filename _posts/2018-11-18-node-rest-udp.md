---
layout: post
title: Node.js HTTP rest server side by side with UDP on same port
tags: [node, http, udp, broadcast, dgram]
---

We are going to build service which will be able to respond to both HTTP requests and UDP messages comming to the same port.

UDP might be used to broadcast messages which might be used to service discovery like things if all services are running in same network.

**server.js**

```js
const http = require("http");
const url = require("url");
const dgram = require("dgram");
const { StringDecoder } = require("string_decoder");

const PORT = process.env.PORT || 3000;

// http/tcp server
const server = http.createServer((req, res) => {
  const method = req.method.toLowerCase();
  const parsed = url.parse(req.url, true);
  const path = parsed.pathname.replace(/^\/+|\/+$/g, "");
  const query = parsed.query;
  const headers = req.headers;

  const decoder = new StringDecoder("utf-8");
  let body = "";
  req.on("data", data => (body += decoder.write(data)));
  req.on("end", () => {
    body += decoder.end();

    console.log({
      kind: "HTTP_REQUEST",
      method,
      path,
      query,
      headers,
      body
    });

    // TODO: process request

    res.setHeader("Content-Type", "application/json; charset=utf-8");
    res.writeHead(200);
    res.end(JSON.stringify({ success: true, message: "ok" }));
  });
});

// udp server
const socket = dgram.createSocket(
  {
    type: "udp4",
    reuseAddr: true // <- NOTE: we are asking OS to let us reuse port
  },
  (buffer, sender) => {
    const message = buffer.toString();
    console.log({
      kind: "UDP_MESSAGE",
      message,
      sender
    });

    // demo: respond to sender
    socket.send(message.toUpperCase(), sender.port, sender.address, error => {
      if (error) {
        console.error(error);
      } else {
        console.log({
          kind: "RESPOND",
          message: message.toUpperCase(),
          sender
        });
      }
    });
  }
);

// POI: bind two servers to same port
server.listen(PORT);
socket.bind(PORT);
```

And here is simple client which will send broadcast udp message to whole network and print response if any, after that it closes socket and exits.

**client.js**

```js
const dgram = require("dgram");
const socket = dgram.createSocket("udp4");

const PORT = process.env.PORT || 3000;

socket.bind();
socket.on("listening", () => {
  socket.setBroadcast(true);

  // 255.255.255.255 - boradcast for local network - RFC922
  socket.send("hi", PORT, "255.255.255.255", err => {
    console.log(err ? err : "Sended");
    // socket.close();
  });

  socket.on("message", (buffer, sender) => {
    const message = buffer.toString();
    console.log(`Received message: ${message} from ${sender.address}`);
    socket.close();
  });
});
```

Now is you start server and try to run following:

```bash
$ node client.js
Sended
Received message: HI from 192.168.105.108
$ curl -X POST http://localhost:6000/foo?acme=42 -d '{"foo":"bar"}'
{"success":true,"message":"ok"}
```

On a server console you will see:

```bash
{ kind: 'UDP_MESSAGE',
  message: 'hi',
  sender:
   { address: '192.168.105.108',
     family: 'IPv4',
     port: 56939,
     size: 2 } }
{ kind: 'RESPOND',
  message: 'HI',
  sender:
   { address: '192.168.105.108',
     family: 'IPv4',
     port: 56939,
     size: 2 } }
{ kind: 'HTTP_REQUEST',
  method: 'post',
  path: 'foo',
  query: { acme: '42' },
  headers:
   { host: 'localhost:3000',
     'user-agent': 'curl/7.54.0',
     accept: '*/*',
     'content-length': '13',
     'content-type': 'application/x-www-form-urlencoded' },
  body: '{"foo":"bar"}' }
```
