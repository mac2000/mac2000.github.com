---
layout: post
title: AmbiguousActionException
tags: [dotnet, AmbiguousActionException]
---

Here is challange, I wish to have following API:

```
GET /persons/1

PATCH /persons/1
{
    "name": "Alex"
}

PATCH /persons/1
{
    "age": 33
}
```

where `PATCH` is used to partially update related entity.

The problem is that it is not so easy to implement in dotnet.

> TLDR: look for code at end of page

If you decide to make two models `PatchPersonName` and `PatchPersonAge` and two appropriate controller actions, something like:

```csharp
public class PatchPersonName
{
    [Required]
    public string Name { get; set; }
}

public class PatchPersonAge
{
    [Required]
    [Range(1, 99)]
    public int Age { get; set; }
}

[ApiController]
public class DemoController : ControllerBase
{
    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    public void PatchPersonName(int id, [FromBody]PatchPersonName model) => {};

    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    public void PatchPersonAge(int id, [FromBody]PatchPersonAge model) => {};
}
```

you immediatelly will catch `AmbiguousActionException` telling you that multiple actions matched so dotnet can not choose the right one.

Next step you might try to do is create some kind of base class class and inherit your models from it. e.g.

```csharp
public class BasePatchModel {}

public class PatchPersonName : BasePatchModel {...}
public class PatchPersonAge : BasePatchModel {...}

[ApiController]
public class DemoController : ControllerBase
{
    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    public void PatchPerson(int id, [FromBody]BasePatchModel model) => {};
}
```

But immediatelly you will get following problem - it wont be deserialized as you expect and will be always empty.

But still you can go further and so something like this:

```csharp
[HttpPatch]
[Route("person/{id:int:min(1)}")]
public object PatchPerson(int id, [FromBody]JObject model)
{...}
```

and now you can use `ToObject<T>` of `JObject` something like this:

```csharp
var patchName = model.ToObject<PatchName>();
if (patchName != null)
{
    ...
}

var patchAge = model.ToObject<PatchAge>();
if (patchAge != null)
{
    ...
}
else
{
    return BadRequest();
}
```

but it becomes ugly and long, also there few more problems you will realize pretty soon.

So first of all - you loose annotation validation :( which can be fixed like this:

```csharp
if (patchName != null && Validator.TryValidateObject(patchName, new ValidationContext(patchName), new List<ValidationResult>(), true))
```

Next, we wish to have separate action methods for further Swashbuckle API documentation generation, but if you try something like:

```csharp
[ApiController]
public class DemoController : ControllerBase
{
    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    public object PatchPerson(int id, [FromBody]JObject model)
    {...}

    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    public void PatchPersonName(int id, [FromBody]PatchPersonName model)
    {...}

    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    public void PatchPersonAge(int id, [FromBody]PatchPersonAge model)
    {...};
}
```

And once again you will get ambiguous error, so what we are going to do is to tell dotnet to always skip our concrete actions with action constraint so dotnet will always use our generic action. And we will hide it from swashbuckle, e.g.

```csharp
public class DoNotActivateAttribute : ActionMethodSelectorAttribute
{
    public override bool IsValidForRequest(RouteContext routeContext, ActionDescriptor action)
    {
        return false;
    }
}

[ApiController]
public class DemoController : ControllerBase
{
    [HttpPatch]
    [ApiExplorerSettings(IgnoreApi = true)] // will hide me from Swashbuckle
    [Route("person/{id:int:min(1)}")]
    public object PatchPerson(int id, [FromBody]JObject model)
    {
        var patchName = model.ToObject<PatchName>();
        if (patchName != null && Validator.TryValidateObject(patchName, new ValidationContext(patchName), new List<ValidationResult>(), true))
        {
            return PatchPersonName(id, patchName);
        }

        var patchAge = model.ToObject<PatchAge>();
        if (patchAge != null && Validator.TryValidateObject(patchAge, new ValidationContext(patchAge), new List<ValidationResult>(), true))
        {
            return PatchPersonAge(id, patchAge);
        }
        else
        {
            return BadRequest();
        }
    }

    [HttpPatch]
    [DoNotActivate] // will skip me when choosing action
    [Route("person/{id:int:min(1)}")]
    public void PatchPersonName(int id, [FromBody]PatchPersonName model)
    {...}

    [HttpPatch]
    [DoNotActivate] // will skip me when choosing action
    [Route("person/{id:int:min(1)}")]
    public void PatchPersonAge(int id, [FromBody]PatchPersonAge model)
    {...};
}
```

So we kind of done, technically it does what we want but the challenge is can it be done other way?!

# Action Method Selector

What we are going to do is to remove generic action, and will try to make our action method selector in a such way that it will try to deserialize incomming request body into desired model and if it will - we are going to select action.

The tricky part here is that we need to enable rewind of request body which is stream. and do not forget to move stream position back to zero when we done - otherwise everything will be broken.

Here is what I end up with:

```csharp
public class PatchForAttribute : ActionMethodSelectorAttribute
{
    public Type Type { get; }

    public PatchForAttribute(Type type)
    {
        Type = type;
    }

    public override bool IsValidForRequest(RouteContext routeContext, ActionDescriptor action)
    {
        // IMPORTANT: required to rewind stream
        routeContext.HttpContext.Request.EnableRewind();
        // kind of copy body stream
        var body = new StreamReader(routeContext.HttpContext.Request.Body).ReadToEnd();
        try
        {
            // we are trying to deserialize incommingn body into desired type
            JsonConvert.DeserializeObject(body, Type, new JsonSerializerSettings { MissingMemberHandling = MissingMemberHandling.Error });
            return true; // yes we can choose this action
        }
        catch (Exception)
        {
            return false; // no we wont choose this action, go further
        }
        finally
        {
            // IMPORTANT: do not forget to rewind stream
            routeContext.HttpContext.Request.Body.Position = 0;
        }
    }
}
```

and now our controller will look like this:

```csharp
[ApiController]
public class DemoController : ControllerBase
{
    [Route("person/{id:int:min(1)}")]
    [PatchFor(typeof(PatchPersonName))]
    public object PatchPersonName(int id, [FromBody]PatchPersonName model)
    {...}

    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    [PatchFor(typeof(PatchPersonAge))]
    public object PatchPersonAge(int id, [FromBody]PatchPersonAge model)
    {...}
}
```

**Pros:** validation is working, no need for third action, will work with swashbuckle

**Cons:** for concrete this actions we are reading and deserializing body twice

# Single route with multiple models

And here is final result, everything combined for a note:

**Program.cs**

```csharp
using System;
using System.ComponentModel.DataAnnotations;
using System.IO;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http.Internal;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Abstractions;
using Microsoft.AspNetCore.Mvc.ActionConstraints;
using Microsoft.AspNetCore.Routing;
using Microsoft.Extensions.DependencyInjection;
using Newtonsoft.Json;

public static class Program
{
    public static void Main(string[] args) => WebHost.CreateDefaultBuilder(args).UseStartup<Startup>().Build().Run();
}

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseMvc();
    }
}

public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}

public class PatchPersonName
{
    [Required]
    public string Name { get; set; }
}

public class PatchPersonAge
{
    [Required]
    [Range(1, 99)]
    public int Age { get; set; }
}

public class PatchForAttribute : ActionMethodSelectorAttribute
{
    public Type Type { get; }

    public PatchForAttribute(Type type)
    {
        Type = type;
    }

    public override bool IsValidForRequest(RouteContext routeContext, ActionDescriptor action)
    {
        routeContext.HttpContext.Request.EnableRewind();
        var body = new StreamReader(routeContext.HttpContext.Request.Body).ReadToEnd();
        try
        {
            JsonConvert.DeserializeObject(body, Type, new JsonSerializerSettings { MissingMemberHandling = MissingMemberHandling.Error });
            return true;
        }
        catch (Exception)
        {
            return false;
        }
        finally
        {
            routeContext.HttpContext.Request.Body.Position = 0;
        }
    }
}

[ApiController]
public class DemoController : ControllerBase
{
    // curl -s http://localhost:5000/person/1 | jq
    // {
    //     "id": 1,
    //     "name": "Alex",
    //     "age": 33
    // }
    [HttpGet]
    [Route("person/{id:int:min(1)}")]
    public Person Person(int id) => new Person
    {
        Id = id,
        Name = "Alex",
        Age = 33
    };

    // curl -s -X PATCH -H 'Content-Type: application/json' http://localhost:5000/person/1 -d '{"name": "Maria"}'
    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    [PatchFor(typeof(PatchPersonName))]
    public object PatchPersonName(int id, [FromBody]PatchPersonName model) => new
    {
        Id = id,
        Kind = "PATCH",
        Prop = "Name",
        Value = model.Name
    };

    // curl -s -X PATCH -H 'Content-Type: application/json' http://localhost:5000/person/1 -d '{"age": 30}' | jq
    // {
    //     "id": 1,
    //     "kind": "PATCH",
    //     "prop": "Age",
    //     "value": 30
    // }
    // # still have working validation
    // curl -s -X PATCH -H 'Content-Type: application/json' http://localhost:5000/person/1 -d '{"age": 30000}' | jq
    // {
    //     "Age": [
    //         "The field Age must be between 1 and 99."
    //     ]
    // }
    [HttpPatch]
    [Route("person/{id:int:min(1)}")]
    [PatchFor(typeof(PatchPersonAge))]
    public object PatchPersonAge(int id, [FromBody]PatchPersonAge model) => new
    {
        Id = id,
        Kind = "PATCH",
        Prop = "Age",
        Value = model.Age
    };
}
```
