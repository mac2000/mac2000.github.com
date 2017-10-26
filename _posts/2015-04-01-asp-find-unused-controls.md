---
layout: post
title: NDepend find unused controls in ASP.NET WebForms site
tags: [asp, webforms, unused, cleanup, ndepend]
---

There is no build in tools in Visual Studio to find unused controls in ASP.NET WebForms site, but there is great third party tool called [ndepend](http://www.ndepend.com/).

At this moment ndepend does not support analyzing of webforms sites but there is one way to perform one.

All you need to do is publish you site to local folder with precompiling all controls all other settings can be leaved with default options.

![Precompile](/images/ndepend/precompile.png)

Now you can use ndepend to analyze assemblies in directory rather than analyzing Visual Studio solution

Notice: If you will get circular reference errors or something like this, try tune precompilation settings like this:

![Precompile settings](/images/ndepend/precompile_settings.png)

It helped me a lot at least to get my project to be compiled

And now here is pretty part - **ndepend is awesome!** it can not only show you 100500 things you event do not know about, but run custom queries over your code

Just look what it can do in response to folloging query:

```csharp
Application.Types.Where(t => t.IsClass && !t.IsGeneratedByCompiler && t.Name.Contains("ascx") && !t.Name.StartsWith("FastObjectFactory")).Select(t => new {
    t,
    childs = t.TypesUsed.Where(p => t.IsClass && !t.IsGeneratedByCompiler && t.Name.Contains("ascx") && !p.Name.StartsWith("FastObjectFactory")),
    parents = t.TypesUsingMe.Where(p => t.IsClass && !t.IsGeneratedByCompiler && !p.Name.StartsWith("FastObjectFactory"))
})
```


WebForms Controls Usage
-----------------------

![WebForms Controls Usage](/images/ndepend/ndepend_webforms_controls_usage.png)

At last there is easy way to find unused controls (but still should be carefull if you add controls dynamically to your page)


WebForms Controls Hierarchy Tree
--------------------------------

Did you ever tried to explain to your PM that project is way to big, there is so many places changes in which can brake something else?

Here is how can you show them picture: Open "Dependency Graph" and call "Export Query Result to Graph" from menu for such query:

```csharp
Application.Types.Where(t => t.IsClass && !t.IsGeneratedByCompiler && (t.Name.Contains("ascx") || t.Name.Contains("aspx")) && !t.Name.StartsWith("FastObjectFactory")).Select(t => new {t})
```

It will look something like this:

![WebForms Controls Hierarchy](/images/ndepend/ndepend_webforms_controls_hierarchy_preview.png)

[Full tree](/images/ndepend/ndepend_webforms_controls_hierarchy.png) *3MB size*

NDepend Users Voice
-------------------

After playing with ndepend created few feature requests, if you are reading this you probably want to do same things as i, so this requests will help you a lot:

[Ability to export dependency graph as SVG](https://ndepend.uservoice.com/forums/226344-ndepend-user-voice/suggestions/7442035-
ability-to-export-dependency-graph-as-svg)

[Support for Web Sites (WebForms/MVC)](https://ndepend.uservoice.com/forums/226344-ndepend-user-voice/suggestions/7442068-support-for-web-sites-webforms-mvc)


Here is how at this moment I am removing unused controls from project:

```csharp
from t in Application.Types
let p = t.TypesUsingMe.Where(p => t.IsClass && !t.IsGeneratedByCompiler && !p.Name.StartsWith("FastObjectFactory"))
where p.Count() == 0 && t.IsClass && !t.IsGeneratedByCompiler && t.Name.Contains("ascx") && !t.Name.StartsWith("FastObjectFactory") && !t.Name.Contains("cvbuilder_popups")
select new{t,p}
```

Find unused methods insibe WebForms Controls
--------------------------------------------

```csharp
from m in Application.Methods
where

    m.NbMethodsCallingMe == 0
&& m.ParentAssembly.Name.Contains("ascx")
&& !m.IsGeneratedByCompiler
&& !m.IsConstructor
&& !m.ParentType.Name.Contains("FastObjectFactory")
&& !m.IsExplicitInterfaceImpl
&& !m.IsClassConstructor
&& !m.IsVirtual

//TODO: events?
&& !m.IsEventAdder
&& !m.IsEventRemover

&& !m.ParentType.IsDelegate

&& !m.Name.StartsWithAny("Page_", "zz")

&& !m.IsPropertyGetter
&& !m.IsPropertySetter

select new { m, m.NbMethodsCallingMe }
```


Find unsued fields inside WebFroms Controls
-------------------------------------------

```csharp
from f in Application.Fields
where f.NbMethodsUsingMe == 0 && f.ParentAssembly.Name.Contains("ascx")
select new { f, f.NbMethodsUsingMe }
```
