---
layout: post
title: ExtJs grid cell tooltip and messagebox

tags: [cell, ext, extjs, grid, messagebox, qtip, quicktip, renderer, sencha, tip, tooltip]
---

Here is code, for displaying tooltip or message box.

```js
function cust(val, meta, record) {
    var hdn1 = record.data.hdn1;
    var hdn2 = record.data.hdn2;

    var icon = (val > 5) ? 'http://rabota.ua/Theme/img/chek_icon.gif' : 'http://rabota.ua/Theme/img/cancel_icon.gif';

    var tip = '<img src=\''+icon+'\' /><br />';
    tip += 'hdn1: '+hdn1+'<br />';
    tip += 'hdn2: '+hdn2+'<br />';
    tip += '<br />';
    tip += '<b>SUM:</b> '+val+'<br />';

    meta.attr = 'ext:qtip="'+tip+'" ext:qtitle="Tip FOR CUSTOM FIELD"';

    var mt = '';
    mt += '&lt;b&gt;Cusomt info&lt;/b&gt;&lt;br /&gt;';
    mt += '&lt;img src=&quot;'+icon+'&quot; /&gt;&lt;br /&gt;';
    mt += '&lt;i&gt;hdn1:&lt;/i&gt; '+hdn1+'&lt;br /&gt;';
    mt += '&lt;i&gt;hdn2:&lt;/i&gt; '+hdn2+'&lt;br /&gt;';
    mt += '&lt;br /&gt;';
    mt += '&lt;b style=&quot;color:red&quot;&gt;SUM:&lt;/b&gt; '+val;

    mt += '&lt;br /&gt;&lt;br /&gt;&lt;small&gt;via &lt;a href=&quot;http://rabota.ua&quot; target=&quot;_blank&quot;&gt;rabota.ua&lt;/a&gt; and yes, mac was here :)&lt;/small&gt;';

    var ret = '<a href="javascript:void(0)" onclick="Ext.MessageBox.alert(\'Info for custom field\', \''+mt+'\');return false;">'+val+'</a>';
    //console.log(record);
    return ret;
}
```

It is renderer used for field that contain sum of two other fields.

Data for this field must be prepared on server, in other case you will not be able to sort\filter field.
