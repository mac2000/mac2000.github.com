---
layout: post
title: Firefox vkontakte share button
permalink: /165
tags: [firefox, javascript, vkontake, share, social]
---

Необходимо добавить новую закладку с слеудющим кодом:

    javascript:(function(){window.open('http://vkontakte.ru/share.php?url='+window.location,'','width=500,height=500')}());