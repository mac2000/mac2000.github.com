---
layout: post
title: Using T4 for JSON fakes
tags: [t4, json, codegeneration]
---

Imagine that you are writing unit tests that heavily depends on json fakes.

Usually what you will do is save json files in your solution and read them as needed.

So everywhere in your tests you will have something like:

```csharp
[TestMethod]
public void TestResponseMappingStarterKit()
{
    var fakeResponse = File.ReadAllText("../../Fakes/Json/FakeResponse.json");
    //...
}
```

What I do not like about this is: Whenever you rename and or move your file things will brake, and there is no way you may fix/refactor tests except dummy find and replace which may brake other things.

There is alternative way - T4.

**Json.tt**

```csharp
<#@ template debug="false" hostspecific="true" language="C#" #>
<#@ assembly name="System.Core" #>
<#@ import namespace="System.Linq" #>
<#@ import namespace="System.Text" #>
<#@ import namespace="System.Collections.Generic" #>
<#@ import namespace="System.IO" #>
<#@ output extension=".cs" #>
namespace Tests.Fakes
{
    public static class Json
    {
<# foreach(var file in Directory.GetFiles(Host.ResolvePath(""), "*.json", SearchOption.AllDirectories)) { #>
        /// <summary>
        /// <#= file.Replace(Host.ResolvePath(""), "").Trim('\\') #>
        /// </summary>
        public static string <#= Path.GetFileNameWithoutExtension(file) #>
        {
            get
            {
                return @"<#= File.ReadAllText(file).Replace("\"", "\"\"") #>";
            }
        }
<# } #>
    }
}
```

So from now on you will have your static `Json` class which contains all fakes, it will look something like this:

```csharp
namespace Tests.Fakes
{
    public static class Json
    {
        /// <summary>
        /// Json\FakeResponse.json
        /// </summary>
        public static string FakeResponse
        {
            get
            {
                return @"{""foo"": ""bar""}";
            }
        }
    }
}
```

and your tests will look like this:

```csharp
[TestMethod]
public void TestResponseMappingStarterKit()
{
    var fakeResponse = Fakes.Json.FakeResponse;
    //...
}
```

Now imagine how much confident you fill be after renaming your fake files. Your tests just wont compile, and you will be able to fix things with build in refactor tools.

BTW there is few more ways achieving this. One of them is to add files as resources. But I personally do not like it, just because I do need add files manually, after removing/renaming them I must reproduce same steps in resources - it does not worth it.
