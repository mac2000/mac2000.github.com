---
layout: post
title: jQuery unwrap links

tags: [child, grabber, html, jquery, link, parser, replacewith, text, unwrap]
---

Two ways:

```js
$('a').contents().unwrap();

$('a').each(function() {
    $(this).replaceWith(this.childNodes);
});
```
