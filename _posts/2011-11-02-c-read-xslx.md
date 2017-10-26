---
layout: post
title: c# read xslx

tags: [xslx excel oledb ole .net c# office]
---

```csharp
string filePath = @"C:\Users\AlexandrM\Dropbox\unon.com.ua\boilers.xlsx";

string conStr = "Provider=Microsoft.ACE.OLEDB.12.0; Data Source=" + filePath + "; Extended Properties=\"Excel 12.0 Xml; HDR=YES\";";

OleDbConnection con = new OleDbConnection(conStr);
OleDbCommand cmd = new OleDbCommand();
cmd.Connection = con;
con.Open();

DataTable dt = con.GetOleDbSchemaTable(OleDbSchemaGuid.Tables, null);
string SheetName = dt.Rows[0]["TABLE_NAME"].ToString();

DataSet ds = new DataSet();
OleDbDataAdapter da = new OleDbDataAdapter();
cmd.CommandText = "SELECT * FROM [" + SheetName + "]";
da.SelectCommand = cmd;
da.Fill(ds);

dataGridView1.DataSource = ds.Tables[0];
con.Close();
```
