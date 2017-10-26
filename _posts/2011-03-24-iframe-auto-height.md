---
layout: post
title: iframe auto height

tags: [auto, crossdomain, height, iframe, javascript, jquery, js]
---

Found at: http://benalman.com/projects/jquery-postmessage-plugin/

Code for iframe `http://site1.com/frame.html`:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>TEST FRAME</title>
<script type="text/javascript" src="http://code.jquery.com/jquery-1.5.min.js"></script>
<script type="text/javascript">(function ($) { var g, d, j = 1, a, b = this, f = !1, h = "postMessage", e = "addEventListener", c, i = b[h] && !$.browser.opera; $[h] = function (k, l, m) { if (!l) { return } k = typeof k === "string" ? k : $.param(k); m = m || parent; if (i) { m[h](k, l.replace(/([^:]+:\/\/[^\/]+).*/, "$1")) } else { if (l) { m.location = l.replace(/#.*$/, "") + "#" + (+new Date) + (j++) + "&" + k } } }; $.receiveMessage = c = function (l, m, k) { if (i) { if (l) { a && c(); a = function (n) { if ((typeof m === "string" && n.origin !== m) || ($.isFunction(m) && m(n.origin) === f)) { return f } l(n) } } if (b[e]) { b[l ? e : "removeEventListener"]("message", a, f) } else { b[l ? "attachEvent" : "detachEvent"]("onmessage", a) } } else { g && clearInterval(g); g = null; if (l) { k = typeof m === "number" ? m : typeof k === "number" ? k : 100; g = setInterval(function () { var o = document.location.hash, n = /^#?\d+&/; if (o !== d && n.test(o)) { d = o; l({ data: o.replace(n, "") }) } }, k) } } } })(jQuery);</script>
<style type="text/css">
html, body {padding:0;margin:0;}
</style>
</head>
<body>
<div style="padding:5px;">
<h4>TEST FRAME</h4>
<p id="cnt">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras gravida mattis metus, id ullamcorper urna euismod vitae. Quisque quis felis vulputate mauris malesuada accumsan vel non mi. Ut fermentum tincidunt fermentum. Fusce pulvinar sagittis augue, a molestie nibh tincidunt ut. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla sodales nulla egestas lacus venenatis dictum euismod tellus suscipit. Proin enim justo, ultricies ac venenatis rutrum, lacinia vitae sapien. Duis pharetra viverra tortor, vehicula mollis sapien euismod quis. Mauris non dui tortor. Maecenas vestibulum vulputate metus, id sollicitudin eros malesuada quis. Donec feugiat aliquam dolor at ornare. Duis at magna nibh. Curabitur vestibulum accumsan volutpat. Donec id sem vitae elit ultricies blandit. In porttitor felis eget metus vulputate semper. Pellentesque suscipit erat nec est consequat laoreet vel nec nulla.</p>
</div>

<script type="text/javascript">
jQuery(document).ready(function($) {
    var parent_url = decodeURIComponent(document.location.hash.replace( /^#/, '' ));
    setInterval(function(){
        $.postMessage(
            { ruacifh: $('body').outerHeight( true ) },
            parent_url,
            parent
        );
    }, 500);

    setInterval(function(){
        var rnd = getRandom(5, 20);
        var cnt = '';
        for(var i = 0; i < rnd; i++) {
            cnt += 'Line ' + (i+1) + '<br />';
        }
        $('#cnt').html(cnt);
    }, 2000);
});

function getRandom(min, max) {
    var randomNum = Math.random() * (max-min);
    return (Math.round(randomNum) + min);
}

</script>
</body>
</html>
```

Code for page `example.com`:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>TEST PAGE</title>
<script type="text/javascript" src="http://code.jquery.com/jquery-1.5.min.js"></script>
<script type="text/javascript">(function ($) { var g, d, j = 1, a, b = this, f = !1, h = "postMessage", e = "addEventListener", c, i = b[h] && !$.browser.opera; $[h] = function (k, l, m) { if (!l) { return } k = typeof k === "string" ? k : $.param(k); m = m || parent; if (i) { m[h](k, l.replace(/([^:]+:\/\/[^\/]+).*/, "$1")) } else { if (l) { m.location = l.replace(/#.*$/, "") + "#" + (+new Date) + (j++) + "&" + k } } }; $.receiveMessage = c = function (l, m, k) { if (i) { if (l) { a && c(); a = function (n) { if ((typeof m === "string" && n.origin !== m) || ($.isFunction(m) && m(n.origin) === f)) { return f } l(n) } } if (b[e]) { b[l ? e : "removeEventListener"]("message", a, f) } else { b[l ? "attachEvent" : "detachEvent"]("onmessage", a) } } else { g && clearInterval(g); g = null; if (l) { k = typeof m === "number" ? m : typeof k === "number" ? k : 100; g = setInterval(function () { var o = document.location.hash, n = /^#?\d+&/; if (o !== d && n.test(o)) { d = o; l({ data: o.replace(n, "") }) } }, k) } } } })(jQuery);</script>
</head>
<body style="background:#ccc;">
<h4>TEST PAGE</h4>
<div id="frm"></div>
<script type="text/javascript">
jQuery(document).ready(function($) {
    var ruacifh = 30;
    var src = 'http://webdiz.com.ua/ift/frame.html#' + encodeURIComponent( document.location.href );

    $('#frm').html('<iframe style="background:#fff" " src="' + src + '" width="300" height="'+ruacifh+'" scrolling="no" frameborder="0"></iframe>');

    $.receiveMessage(
        function(e){
            var h = Number( e.data.replace( /.*ruacifh=(\d+)(?:&|$)/, '$1' ) );
            if ( !isNaN( h ) && h > 0 && h !== ruacifh ) {
                ruacifh = h;
                $('#frm iframe:first').height(ruacifh+'px');
            }
        },
        'http://example.com' //!!! Site where frame is, will be http://rabota.ua
    );
});
</script>
</body>
</html>
```
