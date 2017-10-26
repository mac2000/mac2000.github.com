---
layout: post
title: YUI custom events

tags: [javascript, YUI, yahoo, eventprovider, createevent, fireevent, subscribe]
---

```html
<script type="text/javascript">
    var ep = new YAHOO.util.EventProvider();
    ep.createEvent('CustomEventUserLoggedIn');

    function fe() {
        //TODO: show login form, if login success FireEvent
        ep.fireEvent('CustomEventUserLoggedIn', { login: 'z@z.ua', name: 'hello world' });
    }

    ep.subscribe('CustomEventUserLoggedIn', function (oArgs) {
        alert('custom event detected\r\nDATA:\r\nlogin: '+oArgs.login+'\r\nName: '+oArgs.name);
    });
</script>

<h2>Custom events</h2>
<a href="javascript:void(0)" onclick="fe()">fire custom event &laquo;CustomEventUserLoggedIn&raquo;</a>
```
