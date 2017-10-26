---
layout: post
title: Asp.net ExtJs store response generator

tags: [.net, asp.net, c#, ext, extjs, grid, gridpanel, javascript, js, jsonreader, meta, metadata, reader, store]
---

When using ExtJs grids we always describe the same handlers to retrive data, wich contains some metadata and properties like: success, total etc...

So main idea is to generate all this automaticaly.

Here is class diagram:

![screenshot](/images/wp/15.png)

And here is how now my ashx looks like:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using RabotaUA.Model;
using ExtJs;

namespace RabotaUA.Sales.json.Manager
{
    /// <summary>
    /// Summary description for read
    /// </summary>
    public class read : IHttpHandler
    {

        public void ProcessRequest(HttpContext context)
        {
            context.Response.ContentType = "application/json";

            try
            {
                List<DataInfo> itemsList = CacheHelper.GetCachedList(CacheHelper.CachedList.ManagerList, false);
                ResponseReader<DataInfo> resp = new ResponseReader<DataInfo>(itemsList, itemsList.Count, "Id", "Id", "ASC");
                context.Response.Write(resp);
            }
            catch (Exception e)
            {
                ResponseBase resp = new ResponseBase(false, e.Message);
                context.Response.Write(resp);
            }
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```

Now I dont care if some fields changed etc, all metadata will be regenerated, all I need to do now is to correctly describe ExtJs dataGrid.

Code of classes:

**MetaDataFieldBase**

```csharp
public class MetaDataFieldBase
{
    public string name;
    public string type;
}
```

**MetaDataFieldFactory**

```csharp
public static class MetaDataFieldFactory
{
    public static MetaDataFieldBase Create(string name, string type)
    {
        MetaDataFieldBase field = new MetaDataFieldBase();
        field.name = name;
        field.type = type;

        return field;
    }

    public static MetaDataFieldBase Create(string name, Type type)
    {
        MetaDataFieldBase field = new MetaDataFieldBase();
        field.name = name;
        field.type = TypeMapper.Convert(type);

        return field;
    }

    public static MetaDataFieldBase Create(System.Reflection.PropertyInfo info)
    {
        MetaDataFieldBase field = new MetaDataFieldBase();
        field.name = info.Name;
        field.type = TypeMapper.Convert(info.PropertyType);

        return field;
    }

    public static List<MetaDataFieldBase> CreateListFromType(Type type)
    {
        List<MetaDataFieldBase> fields = new List<MetaDataFieldBase>();

        System.Reflection.PropertyInfo[] propInfo = type.GetProperties(System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.Public);
        foreach (System.Reflection.PropertyInfo info in propInfo)
        {
            if (!info.PropertyType.IsPrimitive && !info.PropertyType.Equals(typeof(string)) && !info.PropertyType.Equals(typeof(DateTime))) continue;

            fields.Add(MetaDataFieldFactory.Create(info));
        }

        return fields;
    }
}
```

**TypeMapper**

```csharp
public static class TypeMapper
{
    public static string Convert(Type type)
    {
        switch (Type.GetTypeCode(type))
        {
            case TypeCode.Byte:
            case TypeCode.Boolean: return "bool";

            case TypeCode.Int16:
            case TypeCode.Int32:
            case TypeCode.Int64:
            case TypeCode.UInt16:
            case TypeCode.UInt32:
            case TypeCode.UInt64:
            case TypeCode.Decimal:
            case TypeCode.SByte:
            case TypeCode.Single:
            case TypeCode.Double: return "number";

            case TypeCode.DateTime: return "date";

            case TypeCode.Char:
            case TypeCode.String: return "string";

            //case TypeCode.Empty:
            //    break;
            //case TypeCode.Object:
            //    break;
            //case TypeCode.DBNull:
            //    break;

            default: return "string";
        }
    }
}
```

**MetaData**

```csharp
public class MetaData
{
    public string idProperty;
    public string root = "data";
    public string totalProperty = "total";
    public string successProperty = "success";
    public string messageProperty = "message";
    public string sortInfo;
    public string direction;
    public List<MetaDataFieldBase> fields;

    public MetaData()
    {
    }

    public MetaData(Type type, string id_column_name, string order_by_column_name, string order_by_direction)
    {
        this.idProperty = id_column_name;
        this.sortInfo = order_by_column_name;
        this.direction = order_by_direction;
        this.fields = MetaDataFieldFactory.CreateListFromType(type);
    }
}
```

**ResponseBase**

```csharp
public class ResponseBase
{
    public bool success = true;
    public string message = "ok";

    public ResponseBase()
    {
        this.success = true;
        this.message = "ok";
    }

    public ResponseBase(bool success, string message)
    {
        this.success = success;
        this.message = message;
    }

    public override string ToString()
    {
        return Newtonsoft.Json.JsonConvert.SerializeObject(this, Newtonsoft.Json.Formatting.Indented, new Newtonsoft.Json.Converters.JavaScriptDateTimeConverter());
    }
}
```

**ResponseReader**

```csharp
public class ResponseReader<T> : ResponseBase
{
    public int total;
    public MetaData metaData;
    public List<T> data;

    public ResponseReader(List<T> items, int total_rows, string id_column_name, string order_by_column_name, string order_by_direction)
    {
        this.total = total_rows;
        this.metaData = new MetaData(typeof(T), id_column_name, order_by_column_name, order_by_direction);
        this.data = items;
    }
}
```

And yep this is not ideal, but hey it works :)
