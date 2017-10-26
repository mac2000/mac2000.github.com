---
layout: post
title: C# LogParser

tags: [.net, asp.net, c#, iis, logparser]
---

Add reference `LogParser.dll`

Add `using MSUtil;`

```csharp
DataSet ds = parseLog(@"SELECT TOP 10 c-ip AS IP, COUNT(*) AS RequestsCount FROM C:\Users\mac\Desktop\ex100811\ex100811.log GROUP BY c-ip ORDER BY RequestsCount DESC");

private DataSet parseLog(string query) {
    LogQueryClassClass logParser = new LogQueryClassClass();
    COMIISW3CInputContextClassClass iisLog = new COMIISW3CInputContextClassClass();

    ILogRecordset rsLP = null;
    ILogRecord rowLP = null;

    rsLP = logParser.Execute(query, iisLog);

    DataTable tab = new DataTable("Results");

    // copy schema
    for (int i = 0; i < rsLP.getColumnCount(); i++)
    {
        DataColumn col = new DataColumn();
        col.ColumnName = rsLP.getColumnName(i);
        switch (rsLP.getColumnType(i)) {
            case 1:
                col.DataType = Type.GetType("System.Int32");
                break;
            case 2:
                col.DataType = Type.GetType("System.Double");
                break;
            case 4:
                col.DataType = Type.GetType("System.DateTime");
                break;
            default:
                col.DataType = Type.GetType("System.String");
                break;
        }
        tab.Columns.Add(col);
    }

    // copy data
    while (!rsLP.atEnd()) {
            rowLP = rsLP.getRecord();
            DataRow row = tab.NewRow();

            for (int i = 0; i < rsLP.getColumnCount(); i++)
            row[i] = Convert.ToString(rowLP.getValue(i));

            tab.Rows.Add(row);
            rsLP.moveNext();
    }

    DataSet ds = new DataSet();
    ds.Tables.Add(tab);

    return ds;
}
```
