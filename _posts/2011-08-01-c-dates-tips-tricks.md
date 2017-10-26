---
layout: post
title: C# Dates tips &#038; tricks

tags: [.net, c#, date, datetime, ranges]
---

```csharp
//Dates range for report last month

System.DateTime reportStartDateTime = new System.DateTime(System.DateTime.Now.AddMonths(-1).Year, System.DateTime.Now.AddMonths(-1).Month, 1, 0, 0, 0);
System.DateTime reportEndDateTime = new System.DateTime(reportStartDateTime.Year, reportStartDateTime.Month, System.DateTime.DaysInMonth(reportStartDateTime.Year, reportStartDateTime.Month), 23, 59, 59);

public static double daysInRange(System.DateTime reportStartDateTime, System.DateTime reportEndDateTime, System.DateTime itemStartDateTime, System.DateTime itemEndDateTime)
{
    System.DateTime s = new System.DateTime(Math.Max(itemStartDateTime.Ticks, reportStartDateTime.Ticks));
    System.DateTime e = new System.DateTime(Math.Min(itemEndDateTime.Ticks, reportEndDateTime.Ticks));

    TimeSpan ts = e - s;

    return ts.TotalDays;
}

DateTime d1 = DateTime.ParseExact("6/23/11", "M/d/yy", System.Globalization.CultureInfo.InvariantCulture);
DateTime d2 = DateTime.ParseExact("2011/07/01", "yyyy/MM/dd", System.Globalization.CultureInfo.InvariantCulture);
```