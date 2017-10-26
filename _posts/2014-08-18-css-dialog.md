---
layout: post
title: CSS Only Dialog
tags: [css, html5, dialog]
---

There is `:target` selector in CSS that is applied to element with same id as url hash.

Here is simple example:

```html
<!DOCTYPE html>
<title>Target Example</title>
<style>
    h1 {color:red}
    h1:target {color:green}
</style>

<h1 id="title">Hello World</h1>
```

If you open this page in browser you will see red text, but if you open page with `#title` hash, text will be green.

So, the idea here is very simple, we will have dialog on page with `display: none` and `id="example"` and link that will point page to `example` hash.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no" />
    <meta charset="UTF-8">
    <title>CSS Only Dialog</title>

    <style>
        body {margin:auto;max-width: 50em;padding:1em}

        .dialog {
            position: fixed;
            top: 0; bottom: 0; left: 0; right: 0;
            background: rgba(0, 0, 0, .5);

            display: none;
        }

        .dialog:target {
            display: block;
        }

        .dialog .container {
            background: white;
            box-sizing: border-box;
            max-width: 90%;
            min-width: 260px;
            padding: 30px;

            position: absolute;

            left: 50%;
            top: 50%;

            -webkit-transform: translatex(-50%) translatey(-50%);
            transform: translatex(-50%) translatey(-50%)
        }

        .dialog a[href="#x"] {
            position: absolute;
            right: 10px;
            top: 10px;

            text-align: center;
            width: 50px;
            font:normal 50px/1 monospace;
            color: #999;
            text-decoration: none;
        }

        .dialog a[href="#ok"] {
            text-decoration: none;
            background: #A700AE;
            color: white;
            display: inline-block;
            padding:.5em 1em;
        }
    </style>
</head>
<body>
    <p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p>
    <p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p>
    <a href="#example">Show Dialog</a>
    <p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p>
    <p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p><p>hello</p>

    <div id="example" class="dialog">
        <div class="container">
            <a href="#x">&times;</a>
            <p>Hello World</p>
            <a href="#ok" class="btn">OK</a>
        </div>
    </div>
</body>
</html>
```

What you should be aware of:

Dialog should be at center of screen - otherwise screen will be "jumping".

Backward/Forward buttons in browser will show/hide dialog again and again - so you should remember about this while developing you app, this is strong and weak part of this technique.

`:target` selector not working in IE8
