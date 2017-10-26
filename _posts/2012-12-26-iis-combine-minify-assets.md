---
layout: post
title: IIS combine and minify assets
tags: [iis, static, cache, assets, minify, bundle, combine]
---

Suppose we have simple site with `/css` and `/js` folders which must be minified and combined.

Create empty web forms web site and install `Microsoft ASP.NET Web Optimization Framework` via NuGet package manager.

Actually all you need from this package is: *Microsoft.AspNet.Web.Optimization.WebForms.dll*, *System.Web.Optimization.dll* and *WebGrease.dll*.

First two needed for bundles to work, last one need for minification.

Add `Global.asax` to your project with content like this:

```csharp
<%@ Application Language="C#" %>
<%@ Import Namespace="System.Web.Optimization" %>

<script runat="server">
    void Application_Start(object sender, EventArgs e)
    {
        BundleTable.EnableOptimizations = true; // this will override Web.config/configuration/system.web/compilation[@debug=true]

        // Example of adding script bundle
        BundleTable.Bundles.Add(new ScriptBundle("~/bundles/js/all").Include(
            "~/js/a.js",
            "~/js/b.js"));

        // Example of adding styles bundle (can be also done via Bundle.config)
        BundleTable.Bundles.Add(new StyleBundle("~/bundles/css/all").Include(
            "~/css/a.css",
            "~/css/b.css"));
    }
</script>
```

Add `Bundle.config` to your project (if needed):

```xml
<?xml version="1.0" encoding="utf-8" ?>
<bundles version="1.0">
    <!-- Example of styles bundle -->
    <styleBundle path="~/bundles/css/config">
    <include path="~/css/a.css" />
    <include path="~/css/b.css" />
    </styleBundle>
    <!-- Scripts bundles can not be done this way -->
</bundles>
```

Make your `Web.config` to look like this:

```xml
<?xml version="1.0"?>
<configuration>
    <system.web>
    <!-- debug="true" turns off bundles - can be overwritten in Global.asax -->
    <compilation debug="false" targetFramework="4.0"/>
    <httpRuntime targetFramework="4.0"/>
    </system.web>
</configuration>
```

Restart your site, and thats all, now you can access your bundles by urls: `/bundles/js/all`, `/bundles/css/all`, `/bundles/css/config`

There is much more can be done with bundles, but in my case i need only such simple example

On IIS 7 *microsoft.web.infrastructure.dll* also needed for proper work.