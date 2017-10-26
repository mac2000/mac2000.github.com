---
layout: post
title: Automate JIRA version management
tags: [jira, rest, api, version, powershell]
---

In our workflow we do use versions in Jira, but there is never ending story, we are forgetting to add new versions, release old and so on.

What do we have is knowledge which version is current.

So here are some ththoughts about automating that.

Example of REST API request from PowerShell
-------------------------------------------

```powershell
$username = 'your_user_name'
$password = '********************************'
$authorization = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("${username}:${password}"))
$headers = @{ Authorization = "Basic $authorization" }

Invoke-RestMethod -Uri http://jira.example.com:85/rest/api/2/project/RUA/versions -Headers $headers | Format-Table projectId, id, name, archived, released, releaseDate -AutoSize
```

Retrieve versions
-----------------

https://docs.atlassian.com/jira/REST/6.1.7/#d2e231

```powershell
Invoke-RestMethod -Uri http://jira.example.com:85/rest/api/2/project/RUA/versions -Headers $headers
```

Create version
--------------

https://docs.atlassian.com/jira/REST/6.1.7/#d2e537

```powershell
$body = @{
	name = '73.8'
	projectId = 10040
}
Invoke-RestMethod -Uri http://jira.example.com:85/rest/api/2/version -Headers $headers -Method Post -ContentType 'application/json' -Body ($body | ConvertTo-Json)
```

Release version
---------------

https://docs.atlassian.com/jira/REST/6.1.7/#d2e607

```powershell
$versionId = 13710
$body = @{
	archived = $false
	released = $true
	releaseDate = (Get-Date -Format 'yyyy-MM-dd')
}
Invoke-RestMethod -Uri "http://jira.example.com:85/rest/api/2/version/$versionId" -Headers $headers-Method Put -ContentType 'application/json' -Body ($body | ConvertTo-Json)
```

Version comprasion
------------------

Powershell actually have special type for versions and can compare them

```powershell
[version]'71.3' -lt [version]'71.5'
```

Create new version if required
------------------------------

```powershell
$ErrorActionPreference = 'Stop'
$CurrentVersion = [version]((Invoke-WebRequest http://test.example.com/version.aspx | Select-Object -ExpandProperty AllElements | Where-Object id -EQ data | Select-Object -ExpandProperty innerHTML) -split ' ' | Select-Object -First 1)
$NextVersionsCount = Invoke-RestMethod -Uri http://jira.example.com:85/rest/api/2/project/RUA/versions -Headers @{ Authorization = "Basic ***************************==" } | Select-Object -ExpandProperty SyncRoot | Where-Object name -Like "$($CurrentVersion.Major).*" | Select-Object *, @{n='version';e={ [version]$_.name }} | Where-Object version -gt $CurrentVersion | Measure-Object | Select-Object -ExpandProperty Count

if($NextVersionsCount -eq 0) {
	$Body = @{
		name = "$($CurrentVersion.Major).$($CurrentVersion.Minor + 1)"
		projectId = 10040
	}
	Invoke-RestMethod -Uri http://jira.rabota.ua:85/rest/api/2/version -Headers @{ Authorization = "Basic ***************************==" } -Method Post -ContentType 'application/json' -Body ($Body | ConvertTo-Json)
}
```

So, from now on responsibility to create new versions in jira is on your machine not by you.
