---
layout: post
title: PowerShell parse RSS

tags: [automate, cmd, powershell, script, shell, rss, parse, webclient, net, encoding, utf8, downloadstring, export-csv]
---

```powershell
$URL = "http://hh.ua/employer.do?showRss=1&employerId="

cls
$wc = new-object net.webclient
$wc.Encoding = [Text.Encoding]::UTF8

for($i = 1; $i -lt 423153; $i++) {
    $ID = $i
    $rss = $url + $id
    $x = [xml]($wc.DownloadString($rss))
    $companyName = ([string]($x.rss.channel.title)).Replace("HeadHunter Vacancies - ", "")
    $companyName
    $f = ("C:\Users\mac\Desktop\res\"+$ID+".csv")
    $x.rss.channel.item | select pubDate, title, link, @{Expression={$companyName}; Label="companyName"} | export-csv -Encoding UTF8 -NoTypeInformation $f
}
```
