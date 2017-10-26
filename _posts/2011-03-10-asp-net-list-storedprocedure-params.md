---
layout: post
title: Asp.Net list storedprocedure params

tags: [.net, asp.net, c#, command, params, procedure, sp, sql, sqlclient, sqlcommand, sqlparameter, storedprocedure]
---

```csharp
System.Data.SqlClient.SqlConnection con = new
System.Data.SqlClient.SqlConnection(Common.ConnectionString);

System.Data.SqlClient.SqlCommand cmd = new System.Data.SqlClient.SqlCommand("spResumeBaseUse_GetList",con);
cmd.CommandType = System.Data.CommandType.StoredProcedure;

con.Open();
System.Data.SqlClient.SqlCommandBuilder.DeriveParameters(cmd);
con.Close();

Response.Write("<table border=\"1\">");
foreach (System.Data.SqlClient.SqlParameter param in cmd.Parameters)
{
    string name = param.ParameterName.ToString().Substring(1);
    if (name == "RETURN_VALUE") continue;

    Response.Write("<tr>");
    Response.Write(string.Format("<td>{0}</td>", name));
    Response.Write(string.Format("<td>{0}</td>", param.SqlDbType));
    Response.Write("</tr>");
}
Response.Write("</table>");
```
