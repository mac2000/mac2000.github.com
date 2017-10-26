---
layout: post
title: PowerShell List Memory Usage by IIS Application Pools
tags: [powershell, ps, iis, pool]
---

Following command:

```powershell
gwmi -ComputerName SRV01, SRV02, SRV03, SRV04 -NS 'root\WebAdministration' -class 'WorkerProcess' | select PSComputerName, AppPoolName,ProcessId , @{n='RAM';e={ [math]::round((Get-Process -Id $_.ProcessId -ComputerName $_.PSComputerName).WorkingSet / 1Mb) }} | sort RAM -Descending | ft -AutoSize
```

Will show iis pools on servers and their memory usage:

```
PSComputerName AppPoolName             ProcessId  RAM
-------------- -----------             ---------  ---
SRV01          RabotaUA 2.0 CMS              376 1443
SRV03          RabotaUA2 Admin             11616 1421
SRV01          RabotaUA 2.0 CMS            13084 1371
SRV02          RabotaUA 2.0                 4564 1260
SRV02          RabotaUA 2.0                11964 1250
SRV02          RabotaUA 2.0                 6044 1245
SRV02          RabotaUA 2.0                 7112 1235
SRV02          RabotaUA 2.0                20352 1173
SRV01          RabotaUA 2.0                13868 1051
SRV01          RabotaUA 2.0                15568 1037
SRV01          RabotaUA 2.0                10180 1018
SRV01          RabotaUA 2.0                13480  997
SRV02          RabotaUA 2.0                19136  964
SRV02          RabotaUA 2.0 SubPortals      1560  935
SRV01          RabotaUA 2.0                 2424  920
SRV02          RabotaUA 2.0 Admin           5724  806
SRV01          RabotaUA 2.0                11960  781
SRV02          ReportServer                15188  726
SRV02          RabotaUA 2.0 Admin           9468  559
SRV01          RabotaUA 2.0 SubPortals      2752  234
SRV01          RabotaUA 2.0 Admin          11096   81
SRV03          RabotaUA2 Admin              9816   51
SRV03          img1                         4944   49
SRV03          js                          10448   30
SRV03          img2                        14552   27
SRV03          css                          8440   25
```
