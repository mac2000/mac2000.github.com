---
layout: post
title: C# user settings

tags: [.net, c#, conf, config, prop, properties, settings, user]
---

Settings are configured in `project properties\settings`.

Do not forget to set User scope for setting or it will be read only.

Code sample:

```csharp
int currentPort = Properties.Settings.Default.Port;
int newPort = currentPort + 1;

Properties.Settings.Default.Port = newPort;
Properties.Settings.Default.Save();

MessageBox.Show(string.Format("WAS: {0}\nNOW: {1}", currentPort, newPort));
```
