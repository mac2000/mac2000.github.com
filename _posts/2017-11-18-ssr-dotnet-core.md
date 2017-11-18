---
layout: post
title: Server Side Rendering with Dotnet Core
tags: [maxmind, bigquery]
---

dotnet core spa services allow you to render any frontend library on backend

detailed instructions can be found [here](https://github.com/aspnet/JavaScriptServices/tree/dev/src/Microsoft.AspNetCore.SpaServices)

the simples possible example:

```
mkdir acme
cd acme
dotnet new mvc
dotnet add package Microsoft.AspNetCore.SpaServices
npm init -y -f
npm i -S aspnet-prerendering
```

few things left:

add following line to `/Views/_ViewImports.cshtml`

```
@addTagHelper *, Microsoft.AspNetCore.SpaServices
```

it will allow you to use `asp-prerender-module` tag helpers in your views

replace `app.UseMvc(...)` in `Startup.cs` with this one:

```csharp
app.UseMvc(routes =>
{
    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");

    routes.MapSpaFallbackRoute(
        name: "spa-fallback",
        defaults: new { controller = "Home", action = "Index" });
});
```

it is actually same only thing is added is `MapSpaFallbackRoute` wich will cach all routes to index view of home controller

now, somewhere ins your `Views/Home/Index.cshtml` place following:

```
<div id="app" asp-prerender-module="src/boot"></div>
```

and add `src/boot.js` with contents like this:

```js
const prerendering = require('aspnet-prerendering');

module.exports = prerendering.createServerRenderer(params => {
    return new Promise((resolve, reject) => {
        const result = `
            <h1>Hello from JS</h1>
            <p>Current time in Node is: ${new Date()}</p>
            <p>Request path is: ${params.location.path}</p>
            <p>Absolute URL is: ${params.absoluteUrl}</p>
        `;
        resolve({html: result})
    });
});
```

and you are ready to go, try to run your app with `dotnet run`

you will see your mvc app as usual but take a look at our demo div which contains content generated on a server side

dotnet core itself has predefined templates for react and angular, but virtually can render anything

as a side note - we have checked and running such projects on dotnet is 3 times faster than node and seems to be more stable under load