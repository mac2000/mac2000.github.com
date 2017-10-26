---
layout: post
title: Yahoo pipes get one item per day

tags: [pipes, rss, yahoo, yahoo_pipes]
---

As example we have very big rss feed and want to get one item per day. All item in feed do not have any pudDate.

![screenshot](/images/wp/117.png)

`dateinput` - will contain default value of start date.

`date formatter` - %s - retruns unix timestamp.

`simple math` - subs both unix times, divides them by 86400 (seconds in day) and ads 1.

`truncate` - cuts feed from begginning to our values (will be 1 - in first day, 2 - second ...)

`tail` - returns last item.

**More complex example**

We have xml like this one:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<items>
    <item>Some text 1</item>
    <item>Some text 2</item>
    <item>Some text 3</item>
    <item>Some text 4</item>
    ...
    <item>Some text 1000</item>
</items>
```

And want to get rss that will retrive one item per day from start date.

![screenshot](/images/wp/28.png)

In this example you may see how to use string regex, generate pseudo random number, work with dates, string, numbers, how to format date, and how to work with loop module.
