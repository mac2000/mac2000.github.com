---
layout: post
title: mysql from c#

tags: [.net, c#, connector, dll, lib, mysql]
---

http://dev.mysql.com/downloads/connector/net/5.2.html

Sample code:

```csharp
MySqlConnectionStringBuilder connBuilder = new MySqlConnectionStringBuilder();
connBuilder.Add("Database", "wp4dev");
connBuilder.Add("Data Source", "localhost");
connBuilder.Add("User Id", "root");
connBuilder.Add("Password", "");

MySqlConnection connection = new MySqlConnection(connBuilder.ConnectionString);
MySqlCommand cmd = connection.CreateCommand();

connection.Open();

//SELECT
StringBuilder tmp = new StringBuilder();
cmd.CommandText = "SELECT * FROM wp_posts";
cmd.CommandType = CommandType.Text;

MySqlDataReader reader = cmd.ExecuteReader();

while (reader.Read())
{
    tmp.AppendLine(String.Format("({0}) {1}",
        reader.GetInt32("ID"),reader.GetString("post_title")
    ));
}

reader.Close();
MessageBox.Show(tmp.ToString());

//MySqlTransaction tran = connection.BeginTransaction();
//MySqlCommand cmd2 = tran.Connection.CreateCommand();

connection.Close();
```

For program work you need add reference to `C:\Program Files(x86)\MySQL\MySQL Connector Net 5.2.7\Binaries\.NET 2.0\MySql.Data.dll`

And do not forget to change `Copy Local` property of reference  to `True`
