---
layout: post
title: Asp.Net проверка MX домена

tags: [.net, admin, asp.net, c#]
---

Частенько в при создании мало мальски больших веб приложений, приходиться возиться с пользователями, в частности с регистрацией, и отсеиванием всякого рода ботов. Поступила задача проверки MX записей домена, email которого пользователь выдает за свой.Да конечно можно отсылать email для подтверждения регистрации – но мы ведь никогда не ищем простых путей.

Дале сам пример:

```csharp
string email = txtEmail.Text;
string[] parts = email.Split('@');
email = parts[parts.Length-1];
string command = "nslookup -type=MX " + email;
try
{
    System.Diagnostics.ProcessStartInfo procStartInfo = new System.Diagnostics.ProcessStartInfo("cmd", "/c " + command);
    procStartInfo.RedirectStandardOutput = true;
    procStartInfo.UseShellExecute = false;
    procStartInfo.CreateNoWindow = true;
    System.Diagnostics.Process proc = new System.Diagnostics.Process();
    proc.StartInfo = procStartInfo;
    proc.Start();
    string result = proc.StandardOutput.ReadToEnd();

    string[] resparts = result.Split(new string[] { "\r\n" }, StringSplitOptions.RemoveEmptyEntries);
    if (resparts.Length == 2 && resparts[0].StartsWith("Server: ") && resparts[1].StartsWith("Address: ")) lblRes.Text = "Domain not exists";
    else if (result.Contains("Non-existent domain")) lblRes.Text = "Domain not exists";
    else if(result.Contains("DNS request timed out")) lblRes.Text = "Timeout";
    else lblRes.Text = "OK";

    lblRes.Text += "<br /><br />" + command;
}
catch (Exception objException)
{
    lblRes.Text = objException.Message;
}
```

Естественно, код "грязный" и требует доработки – но может стать отправной точкой для более изящных решений.
