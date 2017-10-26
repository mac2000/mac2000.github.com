---
layout: post
title: jQuery plugin template

tags: [data, event, jquery, plugin]
---

http://docs.jquery.com/Plugins/Authoring

```js
(function( $ ){
    //default settings
    var settings = {
        'location': 'top',
        'background-color': 'blue'
    };

    var methods = {
        init : function(options) {
            return this.each(function(){
                if (options) {
                    $.extend(settings, options);
                }

                //DO THOME PREPARATIONS HERE
                //settings.location = 'bottom'
                //$(this).data('mykey', 'myval');
            });
        },
        hi : function(to) { //custom method
            //fire custom event
            $(this).trigger('hiEvent', { name : $(this).data('mykey') });
        }
    };

    $.fn.myPlugin = function(options) {
        if (methods[options]) {
            return methods[options].apply(this, Array.prototype.slice.call( arguments, 1 ));
        } else if (typeof options === 'object' || !options) {
            return methods.init.apply(this, arguments);
        } else {
            $.error('Method ' +  options + ' does not exist on jQuery.tooltip');
        }
    };
})(jQuery);

jQuery(document).ready(function($) {
    $('#m1').myPlugin();

    //subscribe to custom event
    $('#m1').bind('hiEvent',function (e, oArgs) {
        alert('custom event detected\r\nDATA:\r\nname: ' + oArgs.name);
    });
});
```
