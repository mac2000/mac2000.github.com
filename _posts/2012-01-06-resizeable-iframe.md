---
layout: post
title: Resizeable iframe

tags: [crossdomain, domain, frame, iframe, javascript, js, postmessage, resize, xd]
---

**In iframe**

```html
<script type="text/javascript" src="http://code.jquery.com/jquery-latest.min.js"></script>
<script type="text/javascript">
    //http://www.onlineaspect.com/2010/01/15/backwards-compatible-postmessage/
    var RUAXD = function () {

        var interval_id,
        last_hash,
        cache_bust = 1,
        attached_callback,
        window = this;

        return {
            postMessage: function (message, target_url, target) {
                if (!target_url) {
                    return;
                }
                target = target || parent;  // default to parent
                if (window['postMessage']) {
                    // the browser supports window.postMessage, so call it with a targetOrigin
                    // set appropriately, based on the target_url parameter.
                    target['postMessage'](message, target_url.replace(/([^:]+:\/\/[^\/]+).*/, '$1'));
                } else if (target_url) {
                    // the browser does not support window.postMessage, so use the window.location.hash fragment hack
                    target.location = target_url.replace(/#.*$/, '') + '#' + (+new Date) + (cache_bust++) + '&' + message;
                }
            },
            receiveMessage: function (callback, source_origin) {
                // browser supports window.postMessage
                if (window['postMessage']) {
                    // bind the callback to the actual event associated with window.postMessage
                    if (callback) {
                        attached_callback = function (e) {
                            if ((typeof source_origin === 'string' && e.origin !== source_origin)
                    || (Object.prototype.toString.call(source_origin) === "[object Function]" && source_origin(e.origin) === !1)) {
                                return !1;
                            }
                            callback(e);
                        };
                    }
                    if (window['addEventListener']) {
                        window[callback ? 'addEventListener' : 'removeEventListener']('message', attached_callback, !1);
                    } else {
                        window[callback ? 'attachEvent' : 'detachEvent']('onmessage', attached_callback);
                    }
                } else {
                    // a polling loop is started & callback is called whenever the location.hash changes
                    interval_id && clearInterval(interval_id);
                    interval_id = null;
                    if (callback) {
                        interval_id = setInterval(function () {
                            var hash = document.location.hash,
                        re = /^#?\d+&/;
                            if (hash !== last_hash && re.test(hash)) {
                                last_hash = hash;
                                callback({ data: hash.replace(re, '') });
                            }
                        }, 100);
                    }
                }
            }
        };
    }();
</script>
<script type="text/javascript">
    var parent_url = decodeURIComponent(document.location.hash.replace(/^#/, ''));
    function send(msg) {
        RUAXD.postMessage(msg, parent_url, parent);
        return false;
    }
</script>
<script type="text/javascript">
    jQuery(document).ready(function ($) {
        //alert($('body').height() + ' hash: ' + window.location.hash);
        send($('body').height());
    });
</script>
```

Notice that jQuery is actualy not nessesary for this script.

**Parent**

```html
<iframe id="RUAFRAME" allowTransparency="1" frameborder="0" scrolling="no" src="http://local.rabota.ua/export/context/company.aspx?ntid=1074" style="width:100%;border:none;"></iframe>
<script type="text/javascript">
var RUAXD = function(){

    var interval_id,
    last_hash,
    cache_bust = 1,
    attached_callback,
    window = this;

    return {
        postMessage : function(message, target_url, target) {
            if (!target_url) {
                return;
            }
            target = target || parent;  // default to parent
            if (window['postMessage']) {
                // the browser supports window.postMessage, so call it with a targetOrigin
                // set appropriately, based on the target_url parameter.
                target['postMessage'](message, target_url.replace( /([^:]+:\/\/[^\/]+).*/, '$1'));
            } else if (target_url) {
                // the browser does not support window.postMessage, so use the window.location.hash fragment hack
                target.location = target_url.replace(/#.*$/, '') + '#' + (+new Date) + (cache_bust++) + '&' + message;
            }
        },
        receiveMessage : function(callback, source_origin) {
            // browser supports window.postMessage
            if (window['postMessage']) {
                // bind the callback to the actual event associated with window.postMessage
                if (callback) {
                    attached_callback = function(e) {
                        if ((typeof source_origin === 'string' && e.origin !== source_origin)
                        || (Object.prototype.toString.call(source_origin) === "[object Function]" && source_origin(e.origin) === !1)) {
                                return !1;
                            }
                            callback(e);
                        };
                    }
                    if (window['addEventListener']) {
                        window[callback ? 'addEventListener' : 'removeEventListener']('message', attached_callback, !1);
                    } else {
                        window[callback ? 'attachEvent' : 'detachEvent']('onmessage', attached_callback);
                    }
                } else {
                    // a polling loop is started & callback is called whenever the location.hash changes
                    interval_id && clearInterval(interval_id);
                    interval_id = null;
                    if (callback) {
                        interval_id = setInterval(function() {
                            var hash = document.location.hash,
                            re = /^#?\d+&/;
                            if (hash !== last_hash && re.test(hash)) {
                                last_hash = hash;
                                callback({data: hash.replace(re, '')});
                            }
                        }, 100);
                    }
                }
            }
    };
}();

var RUADOMAIN = document.getElementById('RUAFRAME').src.replace('http://', '').split('/').shift();
document.getElementById('RUAFRAME').src = document.getElementById('RUAFRAME').src + '#' + encodeURIComponent(document.location.href);

RUAXD.receiveMessage(function(message){
    document.getElementById('RUAFRAME').style.height = parseInt(message.data) + 'px';
    //alert(message.data + ' parent');
}, 'http://' + RUADOMAIN);
</script>
```

Script cant send messages in both directions, between differrent domains.

Found at: http://www.onlineaspect.com/2010/01/15/backwards-compatible-postmessage/

Example: http://onlineaspect.com/uploads/postmessage/parent.html
