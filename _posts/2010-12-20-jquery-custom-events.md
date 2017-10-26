---
layout: post
title: jQuery custom events

tags: [javascript, jquery, events, trigger, bind]
---

```html
<script type="text/javascript">
    function fe() {
        //TODO: show login form, if login success FireEvent
        $('body').trigger('CustomEventUserLoggedIn', { login: 'z@z.ua', name: 'hello world' });
    }
    $('body').bind('CustomEventUserLoggedIn', function (e, oArgs) {
        alert('custom event detected\r\nDATA:\r\nlogin: ' + oArgs.login + '\r\nName: ' + oArgs.name);
    });
</script>
<h2>Custom jQuery events</h2>
<a href="javascript:void(0)" onclick="fe()">fire custom event &laquo;CustomEventUserLoggedIn&raquo;</a>
```
