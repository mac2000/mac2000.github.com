---
layout: post
title: C# string join array of objects

tags: [.net, array, c#, convert, object, string, tostring, convertall]
---

```csharp
string.Join(",", Array.ConvertAll<object, string>((object[])value, Convert.ToString))
```
