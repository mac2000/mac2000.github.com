---
layout: post
title: jQuery counter plugin

tags: [count, counter, jquery, plugin, settimeout, timer]
---

Задача плагина прибавлять приблизительно известное значение по таймеру. По типу как на страницах вКонтакте - где счетчик показывает количество зарегистрировавшихся людей.

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>CV Counter</title>
<script type="text/javascript" src="http://code.jquery.com/jquery-1.5.min.js"></script>
<script type="text/javascript">
/**
 * USAGE:
 * jQuery(document).ready(function($) {
 *      $('#cvcounter').ruacounter({
 *          start_val: 0,    //start value
 *          gain     : 0.784 //gain over 1 sec
 *      });
 *  });
*/
(function($){
    $.fn.extend({
        ruacounter: function(options) {
            var options = $.extend({
                start_val: 0,
                start_timestamp: 1308836057891,
                gain: 0.784
            }, options);

            // get normaly destributed value
            var _normal_distribution = function(mean, std_dev) {
                var x = Math.random() / 2147483647;
                var y = Math.random() / 2147483647;
                var u = Math.sqrt(-2*Math.log(x))*Math.cos(2*Math.PI*y);
                return u * std_dev + mean;
            }

            // get random int
            var _getRandomInt = function(min, max) {
                return Math.floor(Math.random() * (max - min + 1)) + min;
            }

            var _getAddition = function() {
                // get normaly destributed addition
                var add = _normal_distribution(options.gain, 1);

                // fix addition if gain is smaller that 1
                if(options.gain < 1) {
                    add = add / 10;
                }

                return add;
            }

            // run every tick
            var tick = function(el, options) {
                options.start_val += _getAddition();

                el.innerHTML = Math.floor(options.start_val);
            }

            var run = function(el, options) {
                // get normaly redistrebuted delay, close to 1 second
                var delay = Math.floor(_normal_distribution(800, _getRandomInt(10, 100)));
                // run counter
                setTimeout(function() {
                    tick(el, options);
                    run(el, options);
                }, delay);
            }

            return this.each(function() {
                var cycles = Math.floor((new Date().getTime() - options.start_timestamp) / 1000);
                for(var i = 0; i < cycles; i++) {
                    options.start_val += _getAddition();
                }

                options.start_val = Math.floor(options.start_val);

                jQuery(this).html(options.start_val);

                run(this, options);
            });
        }
    });
})(jQuery);

jQuery(document).ready(function($) {
    $('#cvcounter').ruacounter({
        start_val: 0,    //стартовое значение
        gain     : 0.784 //прирост числа за интервал
    });

    $('#cvcounter2').ruacounter({
        start_val: 10,   //стартовое значение
        gain     : 1.784 //прирост числа за интервал
    });
});
</script>
</head>
<body>
<h3>CV Counter1: <span id="cvcounter"></span></h3>
<h3>CV Counter2: <span id="cvcounter2"></span></h3>
</body>
</html>
```
