---
layout: post
title: C# Work with Services

tags: [.net, admin, c#, services]
---

To work with services you need to add reference to `System.ServiceProccess`

Example List Installed Services
-------------------------------

```csharp
ServiceController[] services = ServiceController.GetServices();
StringBuilder sb = new StringBuilder();
foreach (ServiceController service in services) sb.AppendLine(service.ServiceName);
textBox1.Text = sb.ToString();
```

Example Restart Service
-----------------------

http://www.csharp-examples.net/restart-windows-service/

```csharp
ServiceController service = new ServiceController(serviceName);
service.Stop();
service.WaitForStatus(ServiceControllerStatus.Stopped);
service.Start();
service.WaitForStatus(ServiceControllerStatus.Running);
```

To work with services you must have administrator privilegies so in manifest file change requestedExecutionLevel to:

```xml
<requestedExecutionLevel level="requireAdministrator" uiAccess="false" />
```
