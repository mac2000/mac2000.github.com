---
layout: post
title: CSS 3, Media Queries Cheat Sheet

tags: [css, css3, device-width, maximum-scale, media, mediaqueries, min-width, minimum-scale, mobile, orientation, phone, query, tablet, viewport]
---

**Desktop - Tablet - Phone**

```html
<!DOCTYPE HTML>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, minimum-scale=1.0, maximum-scale=1.0" />
    <title>test</title>
    <style type="text/css">
        h3 {text-align:center;}

        /* PRINT VERSION */
        @media print {
            h3:after {content: ' - PRINT'; display: inline;}
        }
        /* PHONE VERSION */
        @media only screen and (min-width: 320px) {
            h3:after {content: ' - PHONE'; display: inline;}
        }
        /* TABLET VERSION */
        @media only screen and (min-width: 768px) {
            h3:after {content: ' - TABLET'; display: inline;}
        }
        /* DESKTOP VERSION */
        @media only screen and (min-width: 980px) {
            h3:after {content: ' - DESKTOP'; display: inline;}
        }
    </style>
</head>
<body>
    <h3>TEST</h3>
</body>
</html>
```

**Desktop - Tablet - Phone With Orientation**

```html
<!DOCTYPE HTML>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, minimum-scale=1.0, maximum-scale=1.0" />
    <title>test2</title>
    <style type="text/css">
        /* GLOBAL STYLES */
        h3 {text-align:center;}

        /* PRINT VERSION */
        @media print {
            h3:after {content: ' - PRINT'; display: inline;}
        }

        /* PHONE LANDSCAPE VERSION */
        @media only screen and (min-width: 320px) and (orientation : landscape) {
            h3:after {content: ' - PHONE LANDSCAPE'; display: inline;}
        }
        /* PHONE PORTRAIT VERSION */
        @media only screen and (min-width: 320px) and (orientation : portrait) {
            h3:after {content: ' - PHONE PORTRAIT'; display: inline;}
        }

        /* TABLET LANDSCAPE VERSION - not work, are the same as desktop */
        @media only screen and (min-width: 768px) and (orientation : landscape) {
            h3:after {content: ' - TABLET LANDSCAPE'; display: inline;}
        }
        /* TABLET PORTRAIT VERSION */
        @media only screen and (min-width: 768px) and (orientation : portrait) {
            h3:after {content: ' - TABLET PORTRAIT'; display: inline;}
        }
        /* DESKTOP VERSION */
        @media only screen and (min-width: 980px) {
            h3:after {content: ' - DESKTOP'; display: inline;}
        }
    </style>
</head>
<body>
    <h3>TEST</h3>
</body>
</html>
```

Tested on iPad2, iPhone3(4), HTC Evo 4g, Nokia C6

Another way, founded at: http://www.studionashvegas.com/design/responsive-design-media-queries/

```html
<!DOCTYPE HTML>
<html lang="en-US">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0" />
<title>Test without orientation</title>
<style>
h3:after {content: ' - DESKTOP AND TABLET LANDSCAPE VERSION'; display: inline;}

@media only screen and (min-width: 768px) and (max-width: 959px) {
    h3:after {content: ' - TABLET PORTRAIT'; display: inline;}
    h3 {color:red;}
}

@media only screen and (max-width: 767px) {
    h3:after {content: ' - PHONE LANDSCAPE AND PORTRAIT VERSION'; display: inline;}
}

/* UNCOMMENT ME TO SPECIFY PHONE PORTRAIT VERSION
@media only screen and (max-width: 479px) {
    h3:after {content: ' - PHONE PORTAIT VERSION'; display: inline;}
}
*/
</style>
</head>
<body>
<h3>TEST</h3>
</body>
</html>
```
