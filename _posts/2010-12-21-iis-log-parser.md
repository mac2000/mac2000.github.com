---
layout: post
title: IIS Log Parser

tags: [admin, iis, log, logparser, s-sitename, s-ip, cs-method, cs-uri-stem, cs-uri-query, s-port, cs-username, c-ip, cs-version, cs-host, sc-status, sc-substatus, sc-win32-status, sc-bytes, cs-bytes, time-taken, timestamp]
---

[Link to download logparser](http://www.microsoft.com/downloads/details.aspx?FamilyID=890cd06b-abf8-4c25-91b2-f8d975cf8c07&displaylang=en)

[Fundction description](http://logparserplus.com/Functions)

**Fields**

```
date time s-sitename s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs-version cs(User-Agent) cs(Referer) cs-host sc-status sc-substatus sc-win32-status sc-bytes cs-bytes time-taken
```

**Export logs to tab separated CSV**

```sh
C:\Program Files (x86)\Log Parser 2.2\LogParser.exe -i:IISW3C -o:TSV "SELECT time,c-ip INTO C:\Users\mac\Desktop\111.csv FROM C:\Users\mac\Desktop\ex100804.log"
```

**Select pages and requests count**

```sql
SELECT cs-uri-stem AS UrlWithoutParams, COUNT(*) AS Total FROM C:\Users\mac\Desktop\ex100802\ex100802.log GROUP BY cs-uri-stem ORDER BY Total DESC
```

**Make timestamp from string**

```sh
LogParser.exe -i:IISW3C "SELECT TIMESTAMP('10/08/04 10:00:00', 'yy/MM/dd hh:mm:ss') FROM C:\Users\mac\Desktop\ex100804.log"
```

**Make timestamp from log date and time**

```sh
LogParser.exe -i:IISW3C "SELECT TO_TIMESTAMP(date, time) FROM C:\Users\mac\Desktop\ex100804.log"
```

**Select logs between two timestamps**

```sh
LogParser.exe -i:IISW3C "SELECT date, time, cs-uri-stem FROM C:\Users\mac\Desktop\ex100804.log WHERE TO_TIMESTAMP(date, time) BETWEEN TIMESTAMP('10/08/04 10:00:00', 'yy/MM/dd hh:mm:ss') AND TIMESTAMP('10/08/04 10:03:00', 'yy/MM/dd hh:mm:ss')"
```

**Select Urls, Requests count between two dates**

```sh
LogParser.exe -i:IISW3C "SELECT cs-uri-stem AS UrlWithoutParams, COUNT(*) AS Total FROM C:\Users\mac\Desktop\ex100804.log WHERE TO_TIMESTAMP(date, time) BETWEEN TIMESTAMP('10/08/04 10:00:00', 'yy/MM/dd hh:mm:ss') AND TIMESTAMP('10/08/04 10:03:00', 'yy/MM/dd hh:mm:ss') GROUP BY cs-uri-stem ORDER BY Total DESC"
```

**Export previous results to CSV**

```sh
LogParser.exe -i:IISW3C -o:TSV "SELECT cs-uri-stem AS UrlWithoutParams, COUNT(*) AS Total INTO C:\Users\mac\Desktop\res.txt FROM C:\Users\mac\Desktop\ex100804.log WHERE TO_TIMESTAMP(date, time) BETWEEN TIMESTAMP('TI10/08/04 10:00:00', 'yy/MM/dd hh:mm:ss') AND TIMESTAMP('10/08/04 10:03:00', 'yy/MM/dd hh:mm:ss') GROUP BY cs-uri-stem ORDER BY Total DESC"
```

**Like previous but filtered static files**

```sh
LogParser.exe -i:IISW3C -o:TSV "SELECT cs-uri-stem AS UrlWithoutParams, COUNT(*) AS Total INTO C:\Users\mac\Desktop\res.txt FROM C:\Users\mac\Desktop\ex100803\ex100803.log WHERE TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) <> 'JPG' AND TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) <> 'JPEG' AND TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) <> 'GIF' AND TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) <> 'PNG' AND TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) <> 'CSS' AND TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) <> 'JS' AND TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) <> 'SWF' AND TO_TIMESTAMP(date, time) BETWEEN TIMESTAMP('10/08/03 10:00:00', 'yy/MM/dd hh:mm:ss') AND TIMESTAMP('10/08/03 14:00:00', 'yy/MM/dd hh:mm:ss') GROUP BY cs-uri-stem ORDER BY Total DESC"
```

**SOME LINKS**

[LogParser — привычный взгляд на непривычные вещи](http://habrahabr.ru/blogs/sql/85758/)

[Logparser and finding information between two dates](http://weblogs.asp.net/steveschofield/archive/2006/12/20/logparser-and-finding-information-between-two-dates.aspx)

Some Examples From Log Parser Lizard

**All on port 80**

```sql
SELECT * FROM #IISW3C# WHERE s-port = 80
```

**Total bytes sent**

```sql
SELECT SUM(sc-bytes) AS TotalKiloBytesSent, Count(*) as TotalHits FROM #IISW3C#
```

**ASP app errors**

```sql
SELECT  EXTRACT_TOKEN(FullUri, 0, '|') AS Uri, EXTRACT_TOKEN(cs-uri-query, -1, '|') AS ErrorMsg, EXTRACT_TOKEN(cs-uri-query, 1, '|') AS LineNo, COUNT(*) AS Total USING STRCAT( cs-uri-stem, REPLACE_IF_NOT_NULL(cs-uri-query, STRCAT('?', cs-uri-query))) AS FullUri
FROM #IISW3C#
WHERE (sc-status = 500) AND (cs-uri-stem LIKE '%.asp')
GROUP BY Uri, ErrorMsg, LineNo
ORDER BY Total DESC
```

**Total by hour**

```sql
SELECT
TO_STRING(time, 'HH') AS Hour,
COUNT(*) AS Hits
FROM c:\tst\ex*.log
GROUP BY Hour
```

**HTTP Status codes**

```sql
SELECT STRCAT(TO_STRING(sc-status), STRCAT('.', TO_STRING(sc-substatus))) AS Status, COUNT(*) AS Total
FROM #IISW3C#
GROUP BY Status
ORDER BY Total DESC
```

**Requests and full status by number of hits**

```sql
SELECT STRCAT(cs-uri-stem,
        REPLACE_IF_NOT_NULL(cs-uri-query, STRCAT('?',cs-uri-query))
        ) AS Request,
    STRCAT(TO_STRING(sc-status),
        STRCAT('.',
            COALESCE(TO_STRING(sc-substatus), '?' )
            )
        ) AS Status,
    COUNT(*) AS Total
FROM #IISW3C#
WHERE (sc-status >= 400)
GROUP BY Request, Status
ORDER BY Total DESC
```

**Hit counts for each extension**

```sql
SELECT TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) AS Extension,
    COUNT(*) AS [Total Hits]
FROM #IISW3C#
GROUP BY Extension
ORDER BY [Total Hits] DESC
```

Total number of bytes generated by each extension

```sql
SELECT TO_UPPERCASE(EXTRACT_EXTENSION( cs-uri-stem )) AS Extension, MUL(PROPSUM(sc-bytes),100.0) AS [Total Bytes]
FROM #IISW3C#
GROUP BY Extension
ORDER BY [Total Bytes] DESC
```

**Distinct users**

```sql
SELECT COUNT(DISTINCT cs-username) AS Users
FROM #IISW3C#
```

**All .htm pages**

```sql
SELECT date, cs-uri-stem
FROM #IISW3C#
WHERE INDEX_OF(cs-uri-stem,'.htm') > -1
```

**All IP Addresses between 192.168.0.0. and 192.168.255.255**

```sql
select c-ip, count(c-ip)
from #IISW3C#
WHERE IPV4_TO_INT(c-ip) BETWEEN IPV4_TO_INT('192.168.0.0') AND IPV4_TO_INT('192.168.255.255')
GROUP BY c-ip
```

**Configure logs**

![screenshot](/images/wp/image02.png)
