---
layout: post
title: Newtonsoft.Json, Linq and KnockoutJs

tags: [.net, c#, javascript, js, json, JsonConvert, JsonConverter, JsonSerializer, knockout, knockoutjs, ko, linq, newtonsoft, observable, observableArray, serialize, WriteJson, WriteRawValue]
---

Here is example of defining custom JsonConverters, which allow control way of serializing objects.

In this code objects are serialized into knockout ko.observable and ko.observableArray.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Test
{
    class Program
    {
        static void Main(string[] args)
        {
            // Dummy data for example, here we have numeric, string and collection values
            var data = new[] {
                new { id = 1, name = "one", nums = new[] {1,2,3} },
                new { id = 2, name = "two", nums = new[] {1,2,3} }
            };

            // Example of linq, notice how we are wrapping variables with ko.observable and ko.observableArray
            var items = from i in data
                        select new
                        {
                            id = new KO.Observable(i.id),
                            name = new KO.Observable(i.name),
                            nums = new KO.ObservableArray(i.nums),
                            cusom = new KO.Wrapped(i, "new SubModel(", ")"),
                            person = new Person() { Name = "Alex", Age = 29, Phones = new List<string>() { "phone1", "phone2" } },
                        };

            Console.WriteLine(KO.JsonConvert.SerializeObject(items));

            Console.ReadKey();
        }

        public class Person
        {
            [Newtonsoft.Json.JsonConverter(typeof(KO.ObservableConverter))]
            public string Name { get; set; }
            [Newtonsoft.Json.JsonConverter(typeof(KO.ObservableConverter))]
            public int Age { get; set; }
            [Newtonsoft.Json.JsonConverter(typeof(KO.ObservableArrayConverter))]
            public List<string> Phones { get; set; }
        }
    }
}

namespace KO
{
    public class Wrapped
    {
        public object Value { get; set; }
        public string Before { get; set; }
        public string After { get; set; }

        public Wrapped(object value = null, string before = null, string after = null)
        {
            Value = value;
            Before = before;
            After = after;
        }
    }

    public class Observable : Wrapped
    {
        public Observable(object value = null) : base(value, "ko.observable(", ")") { }
    }

    public class ObservableArray : Wrapped
    {
        public ObservableArray(object value = null) : base(value, "ko.observableArray(", ")") { }
    }

    public class WrappedConverter : Newtonsoft.Json.JsonConverter
    {
        public override bool CanConvert(Type objectType)
        {
            return (typeof(Wrapped) == objectType || typeof(Wrapped) == objectType.BaseType);
        }

        public override object ReadJson(Newtonsoft.Json.JsonReader reader, Type objectType, object existingValue, Newtonsoft.Json.JsonSerializer serializer)
        {
            return existingValue;
        }

        public override void WriteJson(Newtonsoft.Json.JsonWriter writer, object value, Newtonsoft.Json.JsonSerializer serializer)
        {
            writer.WriteRawValue((value as Wrapped).Before + Newtonsoft.Json.JsonConvert.SerializeObject((value as Wrapped).Value, Newtonsoft.Json.Formatting.None) + (value as Wrapped).After);
        }
    }

    public class ObservableConverter : Newtonsoft.Json.JsonConverter
    {
        public override bool CanConvert(Type objectType)
        {
            return true;
        }

        public override object ReadJson(Newtonsoft.Json.JsonReader reader, Type objectType, object existingValue, Newtonsoft.Json.JsonSerializer serializer)
        {
            return existingValue;
        }

        public override void WriteJson(Newtonsoft.Json.JsonWriter writer, object value, Newtonsoft.Json.JsonSerializer serializer)
        {
            writer.WriteRawValue("ko.observable(" + Newtonsoft.Json.JsonConvert.SerializeObject(value, Newtonsoft.Json.Formatting.None) + ")");
        }
    }

    public class ObservableArrayConverter : Newtonsoft.Json.JsonConverter
    {
        public override bool CanConvert(Type objectType)
        {
            return true;
        }

        public override object ReadJson(Newtonsoft.Json.JsonReader reader, Type objectType, object existingValue, Newtonsoft.Json.JsonSerializer serializer)
        {
            return existingValue;
        }

        public override void WriteJson(Newtonsoft.Json.JsonWriter writer, object value, Newtonsoft.Json.JsonSerializer serializer)
        {
            writer.WriteRawValue("ko.observableArray(" + Newtonsoft.Json.JsonConvert.SerializeObject(value, Newtonsoft.Json.Formatting.None) + ")");
        }
    }

    public static class JsonConvert
    {
        static WrappedConverter wrappedConverter = new WrappedConverter();

        public static string SerializeObject(object value)
        {
            return Newtonsoft.Json.JsonConvert.SerializeObject(value, Newtonsoft.Json.Formatting.Indented, wrappedConverter);
        }
    }
}
```

This code will produce output like this one:

![screenshot](/images/wp/kojson2.png)
