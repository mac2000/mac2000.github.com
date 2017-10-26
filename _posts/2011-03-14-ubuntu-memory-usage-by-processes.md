---
layout: post
title: Ubuntu memory usage by processes

tags: [admin, mem, memory, ps, ram, top, ubuntu]
---

```sh
ps aux | awk '{print $4"\t"$11}' | sort | uniq -c | awk '{print $2" "$1" "$3}' | sort -nr
```
