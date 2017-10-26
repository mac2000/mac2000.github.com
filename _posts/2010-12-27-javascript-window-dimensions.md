---
layout: post
title: Javascript window dimensions

tags: [dimensions, height, javascript, js, width, window]
---

crossbrowser get window dimensions in javascript

```js
var windowWidth = window.innerWidth ? window.innerWidth : document.documentElement.clientWidth ? document.documentElement.clientWidth : document.body.clientWidth;
var windowHeight = window.innerHeight ? window.innerHeight : document.documentElement.clientHeight ? document.documentElement.clientHeight : document.body.clientHeight;
```