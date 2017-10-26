---
layout: post
title: Compass IE filter AlphaImageLoader

tags: [alphaimageloader, compass, css, filter, ie, png, sass, scss, style, styles]
---

```scss
@mixin logo($png) {
    width: image-width('logos/'+$png+'.png');
    height: image-height('logos/'+$png+'.png');
    background:transparent url('http://rabota.ua/Theme/img/logos/'+$png+'.png') no-repeat 0 0;
    -background:none;
    -filter:progid:DXImageTransform.Microsoft.AlphaImageLoader(src='http://rabota.ua/Theme/img/logos/#{$png}.png', sizingMethod='scale');
    /*filter: progid:DXImageTransform.Microsoft.Gradient(GradientType=0, startColorstr='#{$png}', endColorstr='#{$png}');*/
}

.logo {
@include logo('logo_new');
}
```

Will produce:

```css
.logo {
  display: -moz-inline-box;
  -moz-box-orient: vertical;
  display: inline-block;
  vertical-align: middle;
  *vertical-align: auto;
}
.logo {
  *display: inline;
}

.logo {
  width: 328px;
  height: 34px;
  background: transparent url("http://rabota.ua/Theme/img/logos/finance_rabota_ua.png") no-repeat 0 0;
  -background: none;
  -filter: progid:DXImageTransform.Microsoft.AlphaImageLoader(src='http://rabota.ua/Theme/img/logos/finance_rabota_ua.png', sizingMethod='scale');
  /*filter: progid:DXImageTransform.Microsoft.Gradient(GradientType=0, startColorstr='finance_rabota_ua', endColorstr='finance_rabota_ua');*/
}
```