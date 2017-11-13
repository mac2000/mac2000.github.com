---
layout: post
title: Google map custom infoWindow

tags: [geo, gmap, google, javascript]
---

<amp-img src="/images/wp/gmap_custom_infowindow.png" alt="screenshot" width="389" height="281"></amp-img>

Если есть необходимость создать кастомные infoWindow то очень здорово поможет [gmaps-utility-library-dev](http://code.google.com/p/gmaps-utility-library-dev/). А именно их класс [extinfowindow](http://gmaps-utility-library-dev.googlecode.com/svn/tags/extinfowindow/1.2/examples/).

Простенький пример:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8"/>
    <title>ExtInfoWindow Example: Simple Example</title>

    <link type="text/css" rel="Stylesheet" media="screen" href="ym.css"/>

    <script type="text/javascript"  src="http://maps.google.com/maps?file=api&amp;v=2&amp;key=ABQIAAAAjU0EJWnWPMv7oQ-jjS7dYxQGj0PqsCtxKvarsoS-iqLdqZSKfxRdmoPmGl7Y9335WLC36wIGYa6o5Q">
    </script>
    <script src="extinfowindow_packed.js" type="text/javascript"></script>
    <script type="text/javascript">
      var map;
      function load() {
        if (GBrowserIsCompatible()) {
          map = new GMap2(document.getElementById("map"));
          map.setCenter(new GLatLng(44, -96), 4);
          map.addControl(new GLargeMapControl());
          map.enableScrollWheelZoom();

          marker = new GMarker( new GLatLng(44, -96) );

          GEvent.addListener(marker, 'click', function(){
            marker.openExtInfoWindow(
              map,
              "ym",
              "<div class=\"title\">Hello</div><div id=\"z\" class=\"body\">I'm some HTML content that will go in the simple<br>example's ExtInfoWindow.</div>",
              {beakOffset: 3}
            );
          });

          //TODO: if content changes call map.getExtInfoWindow().resize();

          map.addOverlay(marker);
        }
      }
    </script>
  </head>
  <body onload="load()" onunload="GUnload()">
    <div id="map" style="width:400px;height:400px" ></div>
  </body>
</html>
```

И его стили:

```css
#ym{
  width: 300px;
}
#ym_contents{
  background-color: #FFF;
  border: 1px solid  #999;
  -moz-border-radius:3px;-webkit-border-radius:3px;border-radius:3px;
}

#ym_contents .title {
background-color:#FA8A1A;
color:#fff;
}

#ym_beak{
  width: 28px;
  height: 38px;
  background: url('beak.png') top left no-repeat transparent;
}
* html #ym_beak{
  /* Alpha transparencies hack for IE */
  background-image:none;
  filter:progid:DXImageTransform.Microsoft.AlphaImageLoader(src='beak.png', sizingMethod='crop');
}
#ym_tl, #ym_tr, #ym_bl, #ym_br,
#ym_t,#ym_l,#ym_r,#ym_b{
  height: 0px;
  width: 0px;
}

#ym_beak {
    z-index:2;
}

#ym_close{
  width: 21px;
  height: 21px;
  background: url('red_close.png') top left no-repeat transparent;
  cursor: pointer;
}
* html #ym_close{
  background-image:none;
  filter:progid:DXImageTransform.Microsoft.AlphaImageLoader(src='red_close.png', sizingMethod='crop');
}
```
