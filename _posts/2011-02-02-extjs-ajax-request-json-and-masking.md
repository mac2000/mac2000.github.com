---
layout: post
title: ExtJs ajax request, json and masking

tags: [ajax, beforerequest, ext, extjs, javascript, js, json, mask, request, requestcomplete, requestexception, response, responsetext]
---

Links:

http://www.extensions.extjs.com/learn/Ext_FAQ_Ajax

http://snipplr.com/view/9692/extjs--simple-ajax-request-with-callback-and-json-to-object-deserialization/

Code:

```html
<%-- MASKING AJAX --%>
<script type="text/javascript">
    var maskingAjax = new Ext.data.Connection({
        listeners: {
            'beforerequest': {
                fn: function (con, opt) {
                    Ext.get(document.body).mask('Loading...');
                },
                scope: this
            },
            'requestcomplete': {
                fn: function (con, res, opt) {
                    Ext.get(document.body).unmask();
                },
                scope: this
            },
            'requestexception': {
                fn: function (con, res, opt) {
                    Ext.get(document.body).unmask();
                },
                scope: this
            }
        }
    });
</script>

<%-- GET RABOTA.UA COMPANY INFO --%>
<script type="text/javascript">
    function getCompanyInfo(id) {
        maskingAjax.request({
            url: '<%= ResolveUrl("~/json/NotebookCompanyInfo/read.ashx")%>',
            failure: function () { alert('err'); },
            success: function (r, o) {
                var company = Ext.decode(r.responseText);
                alert('all ok'); //reloadDRStore();
            },
            params: { NotebookId: id }
        });
    }
</script>
```
