---
layout: post
title: IE6 iFrame Cookies

tags: [cookies, facebook, fb, ie6, ie7, ie8, iframe, javascript, kb927917]
---

IE6 by default do not save cookies in iFrames.

For example we have two sites main.com and external.com and want to place log in form from main.com on external.com.

Probably the easiest way to do this is to place iframe on external.com that shows lo gin form from main.com, and it works in all browsers except IE6.

To solve this, add:

```csharp
Page.Response.AddHeader("p3p", "CP=\"CAO PSA OUR\"");
```

in page load of lo gin form, so iframe will have enought privileges to save cookies.

Also this helps to fix IE7-8 bugs in facebook iframe applications, whitch throws:

```
HTML Parsing Error: Unable to modify the parent container element before the child element is closed (KB927917)
```

Problem in ScriptResource.axd - which generates javascript functions that tries do something with parent window, whitch is not our.
