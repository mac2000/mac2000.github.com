---
layout: post
title: C# Linq on DataColumnCollection

tags: [.net, c#, linq, columns, cast, datacolumn, where, columnname]
---

```csharp
var q = from x in dt.Columns.Cast<DataColumn>()
    where x.ColumnName != "Month"
    select x.ColumnName;
```
