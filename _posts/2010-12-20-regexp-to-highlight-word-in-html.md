---
layout: post
title: RegExp to highlight word in html

tags: [.net, asp.net, c#, regex, replace, highlight]
---

```csharp
return Regex.Replace(str, @"(<([A-Za-z]+)[^>]*[\>]*)*(" + Parameters.KeyWords + @")\b(.*?)(<\/\\2>)*", string.Format("$1<b style=\"background-color:#ff0;{0}\">$3</b>$4$5", fontweight), RegexOptions.IgnoreCase);
```
