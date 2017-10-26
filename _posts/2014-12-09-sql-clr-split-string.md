---
layout: post
title: SQL CLR Safe split string sample
tags: [sql, crl, split]
---

In Visual Studio create SQL Database Project, add New Item \ SQL CLR C# User Defined Function with follofing code:

```csharp
using System;
using System.Linq; // Requires System.Core to be added to references
using System.Data.SqlTypes;
using Microsoft.SqlServer.Server;
using System.Collections;
using System.Collections.Generic;

public partial class UserDefinedFunctions
{
    [SqlFunction(
        FillRowMethodName = "FillRow",
        TableDefinition = "item INT"
    )]
    public static IEnumerable SafeSplitCommaSeparatedIntegers(SqlString text)
    {
        int result;
        return (text.IsNull ? new string[] { } : text.Value.Split(','))
            .Select(i => int.TryParse(i.Trim(), out result) ? result : SqlInt32.MinValue.Value)
            .Where(i => i != SqlInt32.MinValue.Value).ToArray();
    }

    public static void FillRow(Object input, out SqlInt32 output)
    {
        output = new SqlInt32((int)input);
    }
}
```

**Notice** System.Code should be added to project references for Linq.

Build and publish solution to database and try it:

```sql
SELECT * FROM [dbo].[SafeSplitCommaSeparatedIntegers](' 1 ,2, 3,a,') WHERE item > 1;
```

Will return:

```
+------+
| item |
+------+
| 2    |
| 3    |
+------+
```

More info can be found here: http://msdn.microsoft.com/en-us/library/w2kae45k(v=vs.90).aspx
