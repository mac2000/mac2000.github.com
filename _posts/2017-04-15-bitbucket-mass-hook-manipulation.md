---
layout: post
title: Bitbucket mass hook addition
tags: [bitbucket, powershell, api, hook, slack]
---

Suppose you have many repositories and wish to ensure that each has slack hook with desired url

Here is how you can massively update all your repositories with powershell:


```powershell
$account = 'rabotaua'
$slackDesiredUrl = 'https://hooks.slack.com/services/*********/*********/************************'
$slackHookName = 'Slack'
$username = 'mac2000'
$password = '*******************'

$authorization = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("$($username):$($password)"))
$headers = @{ Authorization = "Basic $authorization" }

$hookBody = @{
	description = $slackHookName
	url = $slackDesiredUrl
	active = $true
	events = @('repo:push')
}

$repositories = @()
$response = @{ next = "https://api.bitbucket.org/2.0/repositories/$account" }
do {
	$response = Invoke-RestMethod -Headers $headers -Uri $response.next
	$repositories += $response.values
} while($response.next)
Write-Host "Retrieved $($repositories.Count) repositories from `"$account`" account" -ForegroundColor Cyan

foreach($repository in $repositories) {
	Write-Host $repository.name -NoNewline
	$hooks = Invoke-RestMethod -Headers $headers -Uri $repository.links.hooks.href
	$slackHook = $hooks.values |? description -EQ $slackHookName | select -First 1
	if ($slackHook) {
		if ($slackHook.url -ne $slackDesiredUrl) {
			Invoke-RestMethod -Method Put -Headers $headers -Uri $slackHook.links.self.href -ContentType 'application/json' -Body ($hookBody | ConvertTo-Json) | Out-Null
			Write-Host " existing hook updated" -ForegroundColor Yellow
		} else {
			Write-Host " nothing to do" -ForegroundColor Cyan
		}
	} else {
		Invoke-RestMethod -Method Post -Headers $headers -Uri $repository.links.hooks.href -ContentType 'application/json' -Body ($hookBody | ConvertTo-Json) | Out-Null
		Write-Host " new hook created" -ForegroundColor Green
	}
}
```
