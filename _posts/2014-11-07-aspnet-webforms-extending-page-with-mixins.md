---
layout: post
title: ASP.NET extending webforms Page with mixins
tags: [asp, net, webforms, page, mixin]
---

It seems that here is how mixins are implemented novadays in .Net:

```csharp
using System;
using System.Web.UI;

public static class HeadlinedPage
{
    public static String Headline = String.Empty;

    public static String GetHeadline(this Page page)
    {
        return Headline;
    }

    public static void SetHeadline(this Page page, String NewHeadline)
    {
        Headline = NewHeadline;
    }
}
```

From now on you can set headline in any user control of your project.
