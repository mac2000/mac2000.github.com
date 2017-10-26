---
layout: post
title: .Net overwrite(extend) array

tags: [.net, asp.net, c#, extend, array, namevaluecollection, linq, allkeys, union]
---

```csharp
NameValueCollection def = new NameValueCollection();

def.Add("p1", "v1");
def.Add("p2", "v2");
def.Add("p3", "v3");

NameValueCollection req = new NameValueCollection();
req.Add("p2", "new-v2");
req.Add("p1", string.Empty);

var x = from i in def.AllKeys.Union(req.AllKeys)
        group i by i into g
        select new { Key = g.Key, Value = (!string.IsNullOrEmpty(req[g.Key])) ? req[g.Key] : def[g.Key] };

foreach (var z in x)
{
    System.Console.WriteLine(z.Value);
}
```
