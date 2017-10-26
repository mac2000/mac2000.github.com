---
layout: post
title: WindowsMobile send SMS

tags: [.net, c#, mobile, .netcf, compacktframework]
---

References
----------

```
Microsoft.WindowsMobile
Microsoft.WindowsMobile.PocketOutlook
```

Code
----

```csharp
using Microsoft.WindowsMobile.PocketOutlook;

try
{
    SmsMessage sms = new SmsMessage();
    sms.Body = "hello world";
    sms.To.Add(new Recipient("+380984561952"));
    sms.Send();
    MessageBox.Show("Message sent!");
}
catch (Exception ex)
{
    MessageBox.Show(ex.Message);
}
```
