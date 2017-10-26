---
layout: post
title: Firefox vkontakte share button

tags: [firefox, javascript, vkontake, share, social]
---

Необходимо добавить новую закладку с слеудющим кодом:

```js
javascript:(function(){window.open('http://vkontakte.ru/share.php?url='+window.location,'','width=500,height=500')}());
```
