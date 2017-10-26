---
layout: post
title: Jquery.UI Widget example
tags: [jquery, ui, widget]
---

Minimal code sample:

```js
(function($, undefined) {
    $.widget('name.space', {
        iAmPrivate:undefined,
        options:{/* default options */},
        _iAmPrivate:function(){},
        iAmPublic:function(){},
        _create:function (){},
        _setOption:function(key, value){
            this._super('_setOption', key, value);
        },
        destroy:function(){
            this._super('destroy');
        }
    });
})(jQuery);
```

Links
-----

[Widget Factory](http://ajpiano.com/widgetfactory/)

[Tips for developing jquery.ui widgets](http://www.erichynds.com/jquery/tips-for-developing-jquery-ui-widgets/)

[Coding your first jquery ui plugin](http://net.tutsplus.com/tutorials/javascript-ajax/coding-your-first-jquery-ui-plugin/)

Example
-------

[mac.message.html](http://mac-blog.org.ua/examples/jquery/ui/mac.message.html)

<iframe src="http://mac-blog.org.ua/examples/jquery/ui/mac.message.html" width="100%" height="300"></iframe>
