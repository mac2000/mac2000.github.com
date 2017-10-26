---
layout: post
title: PowerShell Invoke-SqlCmd ConvertTo-Json
tags: [powershell, invoke-sqlcmd, convertto-json]
---

Here is simple one liner for dumping SQL queries into json

```powershell
Invoke-Sqlcmd -InputFile .\VacanciesExport.sql -ServerInstance 'TESTSRV13' -Database 'Beta' -Username 'sa' -Password $env:TESTSRV13_SQL_PASSWORD | Select-Object * -ExcludeProperty ItemArray, Table, RowError, RowState, HasErrors | ConvertTo-Json | Out-File .\vacancies.json
```

If you will try `ConvertTo-Json` results of `Invoke-SqlCmd` as is you will get bunch of unwanted data like `ItemArray`, `Table` etc. To reduce it we are using: `Select-Object * -ExcludeProperty ItemArray, Table, RowError, RowState, HasErrors`
