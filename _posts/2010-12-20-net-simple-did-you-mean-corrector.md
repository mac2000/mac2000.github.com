---
layout: post
title: .net simple &#171;Did You mean&#187; corrector

tags: [.net, asp.net, c#, mssql, spell, sql]
---

http://www.phpclasses.org/package/4859-PHP-Suggest-corrected-spelling-text-in-pure-PHP.html

```csharp
protected void btnGo_Click(object sender, EventArgs e)
{
    string word = txBxKeyWord.Text;
    string alphabet = "abcdefghijklmnopqrstuvwxyz"; // add cyrillyc letters
    int n = word.Length;
    System.Collections.Generic.List<string> edits = new System.Collections.Generic.List<string>();

    for (int i = 0; i < word.Length; i++)
    {
        edits.Add(word.Substring(0, i) + word.Substring(i + 1));

        foreach (char item in alphabet)
        {
            edits.Add(word.Substring(0, i) + item + word.Substring(i + 1));
        }

    }

    for (int i = 0; i < word.Length - 1; i++)
    {
        edits.Add(word.Substring(0, i) + word[i + 1] + word[i] + word.Substring(i + 2));
    }

    for (int i = 0; i < word.Length + 1; i++)
    {
        foreach (char item in alphabet)
        {
            edits.Add(word.Substring(0, i) + item + word.Substring(i));
        }
    }

    for (int i = 0; i < edits.Count; i++)
    {
        edits[i] = "'" + edits[i] + "'";
    }

    string where = string.Join(",", edits.ToArray());

    System.Data.SqlClient.SqlConnection con = new System.Data.SqlClient.SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings["RabotaUA"].ToString());
    con.Open();

    System.Data.SqlClient.SqlCommand cmd = new System.Data.SqlClient.SqlCommand("SELECT TOP 1 Name FROM Synonymous WHERE Name IN (" + where + ")", con);

    string suggest = (string)cmd.ExecuteScalar();

    con.Close();

    ltRes.Text = string.Format("Did You mean: <b style=\"color:red\">{0}</b>?",suggest);
}
```
