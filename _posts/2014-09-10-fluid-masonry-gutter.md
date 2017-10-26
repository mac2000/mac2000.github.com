---
layout: post
title: Fluid (Responsive) Masonry With Static Gutters
tags: [calc, masonry, fluid, responsive]
---

In my case I want masonry for three dynamic columns with 20px gutter.

Here is example with css `calc` function:

```html
<!DOCTYPE html>
<title>Fluid (Responsive) Masonry With Static Gutters</title>
<style>
body {
    margin: 20px; /* outer gutter */
}

.item {
    background: #ccc;
    margin-bottom: 20px; /* gutter between rows */
    width: calc( (100% - 40.1px) / 3 ); /* columns with - is 1/3 of container with, minus 2 * gutter */
}

.item.h1 {
    height: 100px;
}

.item.h2 {
    height: 150px;
}
</style>
<div id="container">
<div class="item h2"></div>
<div class="item h1"></div>
<div class="item h2"></div>
<div class="item h1"></div>
<div class="item h2"></div>
<div class="item h1"></div>
<div class="item h2"></div>
<div class="item h1"></div>
<div class="item h2"></div>
<div class="item h1"></div>
</div>

<script src="http://masonry.desandro.com/masonry.pkgd.min.js"></script>
<script>
new Masonry(document.getElementById('container'), {gutter: 20});
</script>
```

Works fine in modern (IE 10+) browsers.

`40.1px` notice that `.1px` - it is needed for IE, sometimes after resize it renders all in two columns without it.


And here is way without any functions:

```html
<!DOCTYPE html>
<title>Fluid (Responsive) Masonry With Static Gutters</title>
<style>
body {
    margin: 20px 0 0 20px; /* left and top outer gutter */
}

.item {
    width: 33.3332%; /* column width - 1/3 of container */
}

.gutter {
    padding-right: 20px; /* right column gutter */
    padding-bottom: 20px; /* bottom column gutter */
}

.content {
    background: #ccc;
}

.content.h1 {
    height: 100px;
}

.content.h2 {
    height: 150px;
}
</style>
<div id="container">
<div class="item"><div class="gutter"><div class="content h2"></div></div></div>
<div class="item"><div class="gutter"><div class="content h1"></div></div></div>
<div class="item"><div class="gutter"><div class="content h2"></div></div></div>
<div class="item"><div class="gutter"><div class="content h1"></div></div></div>
<div class="item"><div class="gutter"><div class="content h2"></div></div></div>
<div class="item"><div class="gutter"><div class="content h1"></div></div></div>
<div class="item"><div class="gutter"><div class="content h2"></div></div></div>
<div class="item"><div class="gutter"><div class="content h1"></div></div></div>
<div class="item"><div class="gutter"><div class="content h2"></div></div></div>
<div class="item"><div class="gutter"><div class="content h1"></div></div></div>
</div>

<script src="http://masonry.desandro.com/masonry.pkgd.min.js"></script>
<script>
new Masonry(document.getElementById('container'));
</script>
```

Notice that we set only top and left margin for body, because all three columns will have right gutter, so we do not need right one.
