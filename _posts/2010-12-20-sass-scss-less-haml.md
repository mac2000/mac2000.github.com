---
layout: post
title: Sass, Scss, Less, Haml

tags: [haml, less, ruby, sass, scss]
---

Refactoring with SASS
---------------------

http://chriseppstein.github.com/blog/2010/05/25/refactor-my-stylesheets-digg-edition/

Get Ruby
--------

On Windows

http://rubyinstaller.org/downloads/

http://sass-lang.com/tutorial.html

Less
----

In console:

```sh
gem install less
lessc style.less
```

`style.less` file content:

```less
@brand_color: #4D926F;

#header {
    color: @brand_color;
}

h2 {
    color: @brand_color;
}
```

will be converted to `style.css`:

```css
#header, h2 { color: #4d926f; }
```

Sass
----

```sh
sass C:\Users\mac\Desktop\test.sass:C:\Users\mac\Desktop\test2.css
```

will convert `test.sass` file:

```sass
$blue: #3bbfce
$margin: 16px

.content-navigation
    border-color: $blue
    color: darken($blue, 9%)

.border
    padding: $margin / 2
    margin: $margin / 2
    border-color: $blue
```

to `test2.css`:

```css
.content-navigation {
    border-color: #3bbfce;
    color: #2ca2af;
}

.border {
    padding: 8px;
    margin: 8px;
    border-color: #3bbfce;
}
```

Haml
----

```sh
haml C:\Users\mac\Desktop\test.haml C:\Users\mac\Desktop\test3.html
```

will convert `test.haml` file:

```haml
#content
    .left.column
        %h2 Welcome to our site!
        %p Lorem ipsum
    .right.column
        %p Lorem ipsum
```

to `test3.html`:

```html
<div id='content'>
    <div class='left column'>
        <h2>Welcome to our site!</h2>
        <p>Lorem ipsum</p>
    </div>
        <div class='right column'>
        <p>Lorem ipsum</p>
    </div>
</div>
```

Scss
----

```sh
sass C:\Users\mac\Desktop\test.scss:C:\Users\mac\Desktop\test3.css
```

will convert `test.scss` file:

```scss
$blue: #3bbfce;
$margin: 16px;
.border {
    padding: $margin / 2;
    margin: $margin / 2;
    border-color: $blue;
}
```

to `test3.css`:

```css
.border {
    padding: 8px;
    margin: 8px;
    border-color: #3bbfce;
}
```
