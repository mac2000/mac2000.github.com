---
layout: post
title: ASP.Net MsSQL DataBase CRUD examples

tags: [asp.net, c#, db, mssql]
---

Примеры crud для MsSQL из asp.net

```csharp
SqlConnection con = new SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings["RabotaUA"].ToString());

con.Open();
//simplest
//SqlCommand q = new SqlCommand("SELECT * FROM ChannelEvents", con);
//SqlDataReader r = q.ExecuteReader();

//params
SqlCommand q = new SqlCommand("SELECT * FROM ChannelEvents WHERE EventId = @EventId", con);
SqlParameter param  = new SqlParameter();
param.ParameterName = "@EventId";
param.Value = 1;
q.Parameters.Add(param);
SqlDataReader r = q.ExecuteReader();

//for update, delete
//SqlCommand sqlComm = new SqlCommand("DELETE FROM users WHERE userid=1", sqlConn);
//sqlComm.ExecuteNonQuery();

//return first value from first column
//SqlCommand sqlComm = new SqlCommand("SELECT COUNT(*) FROM users", sqlConn);
//int userCount = (int)sqlComm.ExecuteScalar();

StringBuilder sb = new StringBuilder();
while (r.Read()) {
    for(int i = 0; i < r.FieldCount; i++) {
        sb.AppendLine(r[i].ToString());
        sb.AppendLine("<br />");
    }
    sb.AppendLine("<hr />");
}
r.Close();
con.Close();
return sb.ToString();
```
