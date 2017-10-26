---
layout: post
title: JavaScriptSerializer serrialize any object

tags: [.net, ashx, c#, handler, javascript, javascriptserializer, js, json, serrialize]
---

When object is simple there is no problems and all work like a charm.

Here is some simple example:

```csharp
using System.Web.Script.Serialization;

JavaScriptSerializer js = new JavaScriptSerializer();
string resp = js.Serialize(obj);
```

But when object have some recursive links etc, serializer hungs.

So here is code:

```csharp
NotebookCompanyInfo company = NotebookCompanyDAC.NotebookCompanyGetInfo(NotebookId, 0);

Dictionary<string, object> obj = new Dictionary<string, object>();
System.Reflection.PropertyInfo[] propInfo = company.GetType().GetProperties(System.Reflection.BindingFlags.Instance|System.Reflection.BindingFlags.Public);
foreach (System.Reflection.PropertyInfo info in propInfo) {
    if(!info.PropertyType.IsPrimitive && !info.PropertyType.Equals(typeof(string))) continue;
    try {
        object val = Convert.ChangeType(info.GetValue(company, null), info.PropertyType);
        obj.Add(info.Name, val);
    } catch (Exception) {}
}

//Form responce
string response = "{'success':true,'message':'ok','data':" + js.Serialize(obj) + "}";
```
