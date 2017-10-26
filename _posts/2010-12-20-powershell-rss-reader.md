---
layout: post
title: PowerShell RSS reader

tags: [automate, cmd, powershell, script, webclient, downloadstring, rss, format-table]
---

```powershell
([xml](new-object net.webclient).DownloadString("http://newsrss.bbc.co.uk/rss/newsonline_world_edition/business/rss.xml")).rss.channel.item | format-table title, pubDate
```
