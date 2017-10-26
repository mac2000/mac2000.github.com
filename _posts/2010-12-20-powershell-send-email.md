---
layout: post
title: PowerShell send email

tags: [cmd, powershell, shell, smtpclient, send, mailfrom, mailto, subject, body]
---

```powershell
$emailFrom = "user@yourdomain.com"
$emailTo = "user@yourdomain.com"
$subject = "your subject"
$body = "your body"
$smtpServer = "your smtp server"
$smtp = new-object Net.Mail.SmtpClient($smtpServer)
$smtp.Send($emailFrom, $emailTo, $subject, $body)
```
