---
layout: post
title: jQuery UI widget plugin

tags: [class, jquery, oop, plugin, UI, widget]
---

Sample template for jQuery UI plugins

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Test for jQuery UI widget</title>
<link type="text/css" href="css/ui-lightness/jquery-ui-1.8.13.custom.css" rel="stylesheet" />
<script type="text/javascript" src="js/jquery-1.5.1.min.js"></script>
<script type="text/javascript" src="js/jquery-ui-1.8.13.custom.min.js"></script>

<script type="text/javascript">
(function($) {
    $.widget("mac.Example", {
        options: {
            msg: 'hello',
            onAfterRender: function(evt, data) {
                console.log('onAfterRender triggered : ' + data.theAnswer);
            }
        }

        ,_create: function() {
            this._render();
        }

        ,_render: function() {
            this.element.html(this.options.msg);
            this._trigger("onAfterRender", null, {theAnswer: 42});
        }

        ,option: function(key, value) { //override 'option' public method to call _render after it
            if (value != undefined) {
                this.options[key] = value;
                this._render();
                return this;
            }
            else {
                return this.options[key];
            }
        }

        ,sayHello: function(saying) { //example of public method (private starts with underscore)
            console.log('sayHello called : ' + saying);
        }
    });
})(jQuery);
</script>

<script type="text/javascript">
jQuery(document).ready(function($) {
    $('#tmp').Example();

    console.log($('#tmp').Example('option', 'msg')); //get option value
    $('#tmp').Example('option', 'msg', 'mac was here'); //set option value
    console.log($('#tmp').Example('option', 'msg'));

    $('#tmp').Example('sayHello', 'hello world'); //call public method

    $('#tmp').Example('option', 'onAfterRender', function(evt, data) {
        console.log('NEW onAfterRender triggered : ' + data.theAnswer);
    });
});
</script>
</head>
<body>

<p>Test for jQuery UI widget</p>

<div id="tmp"></div>

<p>world</p>

</body>
</html>
```

Found at http://habrahabr.ru/blogs/javascript/120074/
