---
layout: post
title: CSS - Content before sidebars

tags: [cols, columns, compass, css, float, layout, sass, scss, sidebar, wrapper]
---

Main idea is to have html like this:

```html
<div class="wrapper">
    <div class="content-both-sidebars">CONTENT</div>
    <div class="left">LEFT</div>
    <div class="right">RIGHT</div>
</div>
```

Or like this:

```html
<div class="wrapper">
    <div class="content-left">CONTENT</div>
    <div class="left">LEFT</div>
</div>
```

So browser will display content first while rendering page. It is still semantic. It is good for overall page speed and good user experience while browsing your pages.

Here is example:

![screenshot](/images/wp/129.png)

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <title>sidebar</title>
    <style type="text/css">
        html, body {padding:0;margin:0;background:#eee;}

        .wrapper {
            width:960px;                /* wrapper */
            margin:0 auto;

            overflow:hidden;            /* clearfix */
            *zoom: 1;

            margin-top:1em;             /* colorify */
            background:#fff;
            padding:10px;
        }

        .left {
            float:left;
            width:240px;                /* left sidebar width */

            background:#eee;
        }

        .right {
            float:right;
            width:160px;                /* right sidebar width */

            background:#eee;
        }

        .content-both {
            float:left;
            width:560px;                /* wrapper - left - right */
            margin:0 -800px 0 240px;    /* 0 -1*(wrapper - right) 0 left */

            *display:inline;            /* ie6 double margin fix */
            *zoom: 1;

            background:#ccc;            /* colorify */
        }

        .content-left {
            float:left;
            width:720px;                /* wrapper - left */
            margin:0 -960px 0 240px;    /* 0 -1*wrapper 0 left */

            *display:inline;            /* ie6 double margin fix */
            *zoom:1;

            background:#ccc;            /* colorify */
        }
    </style>
</head>
<body>
    <div class="wrapper">
        <div class="content-both">CONTENT</div>
        <div class="left">LEFT</div>
        <div class="right">RIGHT</div>
    </div>

    <div class="wrapper">
        <div class="content-left">CONTENT</div>
        <div class="left">LEFT</div>
    </div>
</body>
</html>
```

Look at contents div margin calculations.

And here is another example using compass (better show how calculatings are done):

```scss
@import "compass";

$_WRAPPER: 960px;
$_LEFT: 240px;
$_RIGHT: 160px;

.wrapper {
    width:$_WRAPPER;
    margin:0 auto;
    @include clearfix;
}

.left {
    @include float-left;
    width: $_LEFT;
}

.right {
    @include float-right;
    width: $_RIGHT;
}

.content-both-sidebars {
    @include float-left;
    width:$_WRAPPER - $_LEFT - $_RIGHT;
    margin:0 -1*($_WRAPPER - $_RIGHT) 0 $_LEFT;
}

.content-left-sidebar {
    @include float-left;
    width:$_WRAPPER - $_LEFT;
    margin:0 -1*$_WRAPPER 0 $_LEFT;
}
```

Found at: http://www.severnsolutions.co.uk/twblog/archive/2004/07/01/cssnegativemarginsalgebra

Also in drupal zen theme: http://drupal.org/project/zen

Need working liquid solution, for less code, something like this:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
    <title>fluid</title>
    <style type="text/css">
        html, body {padding:0;margin:0;background:#eee;}

        .wrapper {
            width:960px;
            margin:0 auto;

            overflow:hidden;
            zoom:1;

            background:#fff;
            padding:10px;
            margin-top:1em;
        }

        .content {
            float:left;
            width:100%;
        }

        .space-left,
        .space-right {
            margin:0;
            background:#fff;
        }

        .space-left {
            margin-left:150px;
        }

        .space-right {
            margin-right:150px;
        }

        .left {
            float:left;
            width:150px;
            margin-left:-960px;
            background:#ccc;
        }

        .right {
            float:left;
            width:150px;
            margin-left:-150px;
            background:#ccc;
        }
    </style>
</head>
<body>

<div class="wrapper">
    <div class="content">
        <div class="space-left space-right">CONTENT</div>
    </div>
    <div class="left">LEFT</div>
    <div class="right">RIGHT</div>
</div>

<div class="wrapper">
    <div class="content">
        <div class="space-left">CONTENT</div>
    </div>
    <div class="left">LEFT</div>
</div>

<div class="wrapper">
    <div class="content">
        <div class="space-right">CONTENT</div>
    </div>
    <div class="right">RIGHT</div>
</div>

</body>
</html>
```
