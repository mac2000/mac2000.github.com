---
layout: post
title: StackOverflow PowerShell API Clien Sample
tags: [stackoverflow, stackexchange, api, powershell, invoke-restmethod]
---

[Stack Overflow](http://stackoverflow.com/) you guys are awesome, thank you for being open!

First of all there is [StackExchange Data Explorer](http://data.stackexchange.com/) which allow you to run SQL queries like [this](http://data.stackexchange.com/stackoverflow/query/302668) over Stack Overflows databases which is awesome!

Second stuff is [StackExchange API](https://api.stackexchange.com/) which allows you to retrieve some interesting data via REST API.

Here is example of such request and its response:

https://api.stackexchange.com/2.2/tags/synonyms?site=stackoverflow&pagesize=5

```json
{
    "items": [
        {
            "creation_date": 1427427681,
            "applied_count": 0,
            "to_tag": "css",
            "from_tag": "font-weight"
        },
        {
            "creation_date": 1428794822,
            "last_applied_date": 1429012623,
            "applied_count": 6,
            "to_tag": "javascript",
            "from_tag": "javascript-library"
        },
        {
            "creation_date": 1364923625,
            "last_applied_date": 1429037717,
            "applied_count": 1,
            "to_tag": "sql-like",
            "from_tag": "like-operator"
        },
        {
            "creation_date": 1428596908,
            "last_applied_date": 1429223402,
            "applied_count": 37,
            "to_tag": "swift",
            "from_tag": "swift1.2"
        },
        {
            "creation_date": 1285959378,
            "applied_count": 0,
            "to_tag": "each",
            "from_tag": ".each"
        }
    ],
    "has_more": true,
    "quota_max": 300,
    "quota_remaining": 197
}
```

To retrieve full list of synonyms (which can be later used in stuffs like ElasticSearch) you could use following script:

```powershell
$body = @{
    site = 'stackoverflow'
    pagesize = 100
    page = 1
}

if($key) {
    $body['key'] = $key
}

$items = @()

$body['filter'] = 'total'
$response = Invoke-RestMethod -Uri "https://api.stackexchange.com/2.2/tags/synonyms" -Body $body
$pages = $response.total / $body['pagesize']
$body.Remove('filter')

do {
    Write-Progress -Activity 'Retrieving page' -Status $body['page'] -PercentComplete ( [Math]::Min($body['page'] / $pages * 100, 100) )
    $response = Invoke-RestMethod -Uri "https://api.stackexchange.com/2.2/tags/synonyms" -Body $body
    $items += $response.items
    $body['page'] += 1
} while($response.has_more)

$items | select -First 10 | select @{n='From';e={ $_.from_tag.Replace('-', ' ') }}, @{n='To';e={ $_.to_tag.Replace('-', ' ') }}
```

Which will produce output like this:

```
From               To
----               --
font weight        css
javascript library javascript
like operator      sql like
swift1.2           swift
.each              each
column store index columnstore
android ui         android
uber               uber api
elixir lang        elixir
css box model      css
```

While requesting API you should define site data from which you want to retrieve, to get list of available sites you can use something like this:

```powershell
$body = @{
    pagesize = 100
    page = 1
}

if($key) {
    $body['key'] = $key
}

$items = @()

do {
    $response = Invoke-RestMethod -Uri "https://api.stackexchange.com/2.2/sites" -Body $body
    $items += $response.items
    $body['page'] += 1
} while($response.has_more)

$items | where site_type -eq 'main_site' | where site_state -eq 'normal' | select name, api_site_parameter, audience | ft -AutoSize
```

Here is list of "normal" main (not meta) sites:

```
name                           api_site_parameter
----                           ------------------
Stack Overflow                 stackoverflow
Server Fault                   serverfault
Super User                     superuser
Meta Stack Exchange            meta
Web Applications               webapps
Arqade                         gaming
Webmasters                     webmasters
Seasoned Advice                cooking
Game Development               gamedev
Photography                    photo
Cross Validated                stats
Mathematics                    math
Home Improvement               diy
Geographic Information Systems gis
TeX - LaTeX                    tex
Ask Ubuntu                     askubuntu
Personal Finance &amp; Money   money
English Language &amp; Usage   english
Stack Apps                     stackapps
User Experience                ux
Unix &amp; Linux               unix
WordPress Development          wordpress
Theoretical Computer Science   cstheory
Ask Different                  apple
Role-playing Games             rpg
Bicycles                       bicycles
Programmers                    programmers
Electrical Engineering         electronics
Android Enthusiasts            android
Physics                        physics
Information Security           security
Graphic Design                 graphicdesign
Database Administrators        dba
Science Fiction &amp; Fantasy  scifi
Skeptics                       skeptics
Drupal Answers                 drupal
SharePoint                     sharepoint
Mi Yodeya                      judaism
Travel                         travel
Christianity                   christianity
Movies &amp; TV                movies
Mathematica                    mathematica
Academia                       academia
The Workplace                  workplace
Salesforce                     salesforce
ExpressionEngine&#174; Answers expressionengine
MathOverflow                   mathoverflow.net
```

To be able to make more requests you should [register](http://stackapps.com/apps/oauth/register) your app.

Unfortunatelly there is not API for careers but there is [feed](http://careers.stackoverflow.com/jobs/feed) that contains interesting information, here is few samples what can be done:

Firs of all we need retrieve feed itself and process it:

```powershell
$xml = [xml](Invoke-WebRequest http://careers.stackoverflow.com/jobs/feed)

$items = @()
foreach($item in $xml.rss.channel.item) {
    if($item.category) {
        $parts = $item.title.Substring($item.title.LastIndexOf('(')).Trim(@(' ', '(', ')')).Split(',')

        $city = $parts | select -First 1
        $region = $parts | select -Last 1

        foreach($category in $item.category) {
            $el = New-Object psobject
            $el | Add-Member NoteProperty 'City' $city.Trim()
            $el | Add-Member NoteProperty 'Region' $region.Trim()
            $el | Add-Member NoteProperty 'Category' $category
            $items += $el
        }
    }
}
```

Now we can get some aggregated data

**Top 10 vacancies by region**

```powershell
$items | Group-Object Region | select Name, Count | Sort-Object Count -Descending | select -First 10 | ft -AutoSize
```

![Top 10 vacancies by region](http://goo.gl/qFJBEv)

```
Name          Count
----          -----
UK              512
CA              479
Deutschland     373
allows remote   291
NY              290
Netherlands     195
Germany         190
WA              123
Canada          114
Sweden           95
```

**Top 10 vacancies by category**

```powershell
$items | Group-Object Category | select Name, Count | Sort-Object Count -Descending | select -First 10 | ft -AutoSize
```

![Top 10 vacancies by region](http://goo.gl/8Y3Tzc)

```
Name       Count
----       -----
java         246
javascript   223
python       128
c#           108
php          107
c++          103
sql           84
linux         79
mysql         73
css           70
```

**Top 10 vacancies by both region and category**

```powershell
$items | Group-Object Region, Category | Sort-Object Count -Descending | select @{n='Region';e={ $_.Values[0] }}, @{n='Category';e={ $_.Values[1] }}, Count | select -First 10 | ft -AutoSize
```

```
Region        Category   Count
------        --------   -----
UK            java          34
UK            javascript    33
CA            java          27
CA            javascript    26
NY            java          23
Deutschland   java          23
allows remote javascript    22
CA            python        22
UK            c#            20
UK            php           20
```


Google Pie chart can be made from PowerShell with that piece of code:

```powershell
$chart = $items | Group-Object Region | select Name, Count | Sort-Object Count -Descending | select -First 5
$chd = 't:' + (($chart | select -ExpandProperty Count) -join ',')
$chl=($chart | select @{n='Label';e={ $_.Name + ' - ' + $_.Count }} | select -ExpandProperty Label) -join '|'
"https://chart.googleapis.com/chart?cht=p&chs=600x200&chd=$chd&chl=$chl"
```
