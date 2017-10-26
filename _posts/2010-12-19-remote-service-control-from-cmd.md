---
layout: post
title: remote service control from cmd

tags: [cmd, iis, remote, shell]
---

```sh
net use \\server password /USER:user
sc \\betasrv stop W3SVC
```
