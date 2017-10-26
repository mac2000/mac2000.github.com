---
layout: post
title: Check what PID is listening port

tags: [admin, administration, cmd, ip, netstat, network, port, powershell, ps, shell]
---

```sh
netstat -o -n -a | findstr 0.0:80
```

And here is more better way with powershell

```powershell
$pids_on_80 = netstat -o -n -a | select-string -pattern ".*:80.*LISTENING.*" | % { $_.Matches } | % { $_.Value.split(" ")[-1] }
Get-Process | ? { $pids_on_80 -contains $_.Id}  | select Id, ProcessName, Path
```
