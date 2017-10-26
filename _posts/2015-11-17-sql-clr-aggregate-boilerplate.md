---
layout: post
title: SQL Server CLR Aggregate boilerplate
tags: [sql, clr, aggregate, median, boilerplate, sample]
---

Boilerplate sample of custom aggregate CLR for SQL Server

First of all you will need ClassLibrary project created (in my case "Median"). With code like this:

```csharp
using Microsoft.SqlServer.Server;
using System;
using System.Collections.Generic;
using System.Data.SqlTypes;
using System.Linq;
using System.IO;

[Serializable]
[SqlUserDefinedAggregate(
		Format.UserDefined,              // because of List<double>
		IsInvariantToDuplicates = false, // receiving the same value again changes the result
		IsInvariantToNulls = false,      // receiving a NULL value changes the result
		IsInvariantToOrder = true,       // the order of the values doesn't affect the result
		IsNullIfEmpty = true,            // if no values are given the result is null
		MaxByteSize = -1                 // without this thing you will get "The size (0) for "Median.Median" is not in the valid range. Size must be -1 or a number between 1 and 8000." error
)]
public struct Median : IBinarySerialize
{
	public SqlDouble Terminate()
	{
		if (collection.Count % 2 == 0)
		{
			return collection.OrderBy(i => i).Skip(-1 + collection.Count / 2).Take(2).Average();
		}
		else
		{
			return collection.OrderBy(i => i).Skip(collection.Count / 2).FirstOrDefault();
		}
	}

	#region Boilerplate
	private List<double> collection;

	public void Init()
	{
		collection = new List<double>();
	}

	public void Accumulate(SqlDouble number)
	{
		collection.Add(number.Value);
	}

	public void Merge(Median mymed)
	{
		collection.AddRange(mymed.collection);
	}

	public void Read(BinaryReader reader)
	{
		var count = reader.ReadInt32();
		collection = new List<double>();
		for (var i = 0; i < count; i++)
		{
			collection.Add(reader.ReadDouble());
		}
	}

	public void Write(BinaryWriter writer)
	{
		writer.Write(collection.Count);
		foreach (var item in collection)
		{
			writer.Write(item);
		}
	}
	#endregion
}
```

Notice that we do not wrap our class into namespace, you may do it but later on while importing to SQL you should do not forget about it.

You may call your class as you wish, it is not required to have same name, while adding your clr to sql you will define by hand which classes from which dlls should be imported.

My first attempt was to use just `[SqlUserDefinedAggregate(Format.UserDefined)]` but I immediatelly got `The size (0) for "Median.Median" is not in the valid range. Size must be -1 or a number between 1 and 8000.` error.

If you with to work with other types of data than doubles you should replace it in boilerplate region.

If your code uses something like `List<int>` you must implement `IBinarySerialize`. In our case all we have as simple as possible boilerplate implementation.

Main logic of your aggregation is in `Terminate` method. It may be not best idea especially if you working with really huge amount of values.

And here is how to use it from SQL:

```sql
-- STEP 1. Enable CLR
sp_configure 'clr enabled', 1
GO
RECONFIGURE
GO

-- STEP 2. reCREATE assembly and function
IF EXISTS (SELECT * FROM sys.objects WHERE name = 'Median') DROP AGGREGATE Median
GO

IF EXISTS (SELECT * FROM sys.assemblies WHERE name = 'Median') DROP ASSEMBLY Median
GO

CREATE ASSEMBLY Median FROM 'C:\Users\Alexandr\Documents\visual studio 2015\Projects\Median\Median\bin\Debug\Median.dll'
GO

CREATE AGGREGATE [dbo].[Median] (@number [float]) RETURNS [float] EXTERNAL NAME [Median].[Median]
GO

-- Step 3. Sample
CREATE TABLE #Temp
(
	Salary FLOAT
);
INSERT INTO #Temp VALUES (10), (11), (9), (20), (5);
SELECT dbo.Median(Salary) FROM #Temp;
DROP TABLE #Temp;
GO
```

Note that `[dbo].[Median]` may be changed to anything you want.

While creating assembly do not forget to fix path to DLL file, and take a note that you can call your assembly anyway you want, but should use same name when creating aggregation function.

`EXTERNAL NAME [Median].[Median]` should be read as `EXTERNAL NAME [my_assembly_name].[my_class_name]` and if you wrap your class with namespace it should be something like `EXTERNAL NAME [my_assembly_name].[my_namespace].[my_class_name]`.


You may also wish to look at example of [Safe split string](http://mac-blog.org.ua/sql-clr-split-string/) and [XSLT inside SQL](http://mac-blog.org.ua/sql-xml-csharp/#xslt-your-xml-right-inside-sql)
