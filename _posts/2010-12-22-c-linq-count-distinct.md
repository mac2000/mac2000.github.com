---
layout: post
title: C# linq count distinct

tags: [.net, c#, linq]
---

Here is simple example:

```csharp
//var list = new List<string> { "a", "b", "a", "c", "a", "b" };
var list = new object[] { "a", "b", "a", "c", "a", "b" };
var q = from x in list
    group x by x into g
    let count = g.Count()
    orderby count descending
    select new { Value = g.Key, Count = count };

StringBuilder sb = new StringBuilder();
foreach (var x in q) {
    sb.AppendLine("Value: " + x.Value + " Count: " + x.Count);
}
MessageBox.Show(sb.ToString());
```

Чуть более сложный пример:

```csharp
public class slice
{
    public slice(string Browser, int BrowserVersion, int PageViews)
    {
        browser = Browser;
        browserVersion = BrowserVersion;
        pageViews = PageViews;
    }
    public string browser;
    public int browserVersion;
    public int pageViews;
}

List<slice> list = new List<slice>();
list.Add(new slice("Opera", 9, 1));
list.Add(new slice("Opera", 9, 1));
list.Add(new slice("Opera", 10, 1));
list.Add(new slice("Chrome", 6, 1));
list.Add(new slice("Chrome", 6, 1));
list.Add(new slice("Chrome", 7, 1));

var q = from x in list
    group x by new { x.browser, x.browserVersion } into g
    select new slice(g.Key.browser, g.Key.browserVersion, g.Sum(s => s.pageViews));

StringBuilder sb = new StringBuilder();
foreach (var x in q) {
    sb.AppendLine("browser: " + x.browser + " browserVersion: " + x.browserVersion+" pageViews: "+x.pageViews);
}

MessageBox.Show(sb.ToString());
```

На выходе даст текст:

```
browser: Opera browserVersion: 9 pageViews: 2
browser: Opera browserVersion: 10 pageViews: 1
browser: Chrome browserVersion: 6 pageViews: 2
browser: Chrome browserVersion: 7 pageViews: 1
```

В этом примере мы сгруппировали записи и посчитали суммы для pageViews.
