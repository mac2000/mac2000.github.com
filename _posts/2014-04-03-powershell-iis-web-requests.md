---
layout: post
title: PowerShell List IIS Web Requests
tags: [powershell, ps, iis]
---

Following command:

```powershell
Get-WebRequest | select timeElapsed, timeInState, timeInModule, currentModule, url, @{n='pipeLineState';e={ switch($_.Attributes | ?{ $_.Name -eq 'pipeLineState'} | select -ExpandProperty Value) { 1 {'BeginRequest'} 2 {'AuthenticateRequest'} 4 {'AuthorizeRequest'} 8 {'ResolveRequestCache'} 16 {'MapRequestHandler'} 32 {'AcquireRequestState'} 64 {'PreExecuteRequestHandler'} 128 {'ExecuteRequestHandler'} 256 {'ReleaseRequestState'} 512 {'UpdateRequestCache'} 1024 {'LogRequest'} 2048 {'EndRequest'} 536870912 {'SendResponse'} default {'Unknown'} } }} | Sort-Object timeElapsed -Descending | ft timeElapsed, timeInState, timeInModule, currentModule, pipeLineState, url -AutoSize
```

Will not only show current running requests, but also therir pipe line state
