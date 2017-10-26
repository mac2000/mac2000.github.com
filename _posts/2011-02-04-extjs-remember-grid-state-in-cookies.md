---
layout: post
title: Extjs remember grid state in cookies

tags: [cols, columns, cookie, cookies, ext, extjs, grid, gridpanel, state, statemanager, stateprovider]
---

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Remember grid state</title>
<link rel="stylesheet" type="text/css" href="library/ext/ext-3.3.0/resources/css/ext-all.css" />
<script type="text/javascript" src="library/ext/ext-3.3.0/adapter/ext/ext-base.js"></script>
<script type="text/javascript" src="library/ext/ext-3.3.0/ext-all.js"></script>
<!-- STATE PROVIDER VIA COOKIES -->
<script type="text/javascript">
Ext.state.Manager.setProvider(new Ext.state.CookieProvider({
    expires: new Date(new Date().getTime()+(1000*60*60*24*7)), //7 days from now
}));
</script>
<!-- LOCAL DATA -->
<script type="text/javascript">
var myData = [
    ['3m Co',                               71.72, 0.02,  0.03,  '9/1 12:00am'],
    ['Alcoa Inc',                           29.01, 0.42,  1.47,  '9/1 12:00am'],
    ['Altria Group Inc',                    83.81, 0.28,  0.34,  '9/1 12:00am'],
    ['American Express Company',            52.55, 0.01,  0.02,  '9/1 12:00am'],
    ['American International Group, Inc.',  64.13, 0.31,  0.49,  '9/1 12:00am'],
    ['AT&T Inc.',                           31.61, -0.48, -1.54, '9/1 12:00am'],
    ['Boeing Co.',                          75.43, 0.53,  0.71,  '9/1 12:00am'],
    ['Caterpillar Inc.',                    67.27, 0.92,  1.39,  '9/1 12:00am'],
    ['Citigroup, Inc.',                     49.37, 0.02,  0.04,  '9/1 12:00am'],
    ['E.I. du Pont de Nemours and Company', 40.48, 0.51,  1.28,  '9/1 12:00am'],
    ['Exxon Mobil Corp',                    68.1,  -0.43, -0.64, '9/1 12:00am'],
    ['General Electric Company',            34.14, -0.08, -0.23, '9/1 12:00am'],
    ['General Motors Corporation',          30.27, 1.09,  3.74,  '9/1 12:00am'],
    ['Hewlett-Packard Co.',                 36.53, -0.03, -0.08, '9/1 12:00am'],
    ['Honeywell Intl Inc',                  38.77, 0.05,  0.13,  '9/1 12:00am'],
    ['Intel Corporation',                   19.88, 0.31,  1.58,  '9/1 12:00am'],
    ['International Business Machines',     81.41, 0.44,  0.54,  '9/1 12:00am'],
    ['Johnson & Johnson',                   64.72, 0.06,  0.09,  '9/1 12:00am'],
    ['JP Morgan & Chase & Co',              45.73, 0.07,  0.15,  '9/1 12:00am'],
    ['McDonald\'s Corporation',             36.76, 0.86,  2.40,  '9/1 12:00am'],
    ['Merck & Co., Inc.',                   40.96, 0.41,  1.01,  '9/1 12:00am'],
    ['Microsoft Corporation',               25.84, 0.14,  0.54,  '9/1 12:00am'],
    ['Pfizer Inc',                          27.96, 0.4,   1.45,  '9/1 12:00am'],
    ['The Coca-Cola Company',               45.07, 0.26,  0.58,  '9/1 12:00am'],
    ['The Home Depot, Inc.',                34.64, 0.35,  1.02,  '9/1 12:00am'],
    ['The Procter & Gamble Company',        61.91, 0.01,  0.02,  '9/1 12:00am'],
    ['United Technologies Corporation',     63.26, 0.55,  0.88,  '9/1 12:00am'],
    ['Verizon Communications',              35.57, 0.39,  1.11,  '9/1 12:00am'],
    ['Wal-Mart Stores, Inc.',               45.45, 0.73,  1.63,  '9/1 12:00am']
];
</script>
<!-- STORE -->
<script type="text/javascript">
var store = new Ext.data.ArrayStore({
    fields: [
        {name: 'company'},
        {name: 'price',      type: 'float'},
        {name: 'change',     type: 'float'},
        {name: 'pctChange',  type: 'float'},
        {name: 'lastChange', type: 'date', dateFormat: 'n/j h:ia'}
    ]
});
store.loadData(myData);
</script>
<!-- GRID -->
<script type="text/javascript">
var grid = new Ext.grid.GridPanel({
        id: 'grid', //need for state
        region: 'center',
        viewConfig: { forceFit: true },
        store: store,
        colModel: new Ext.grid.ColumnModel({
            defaults: { sortable: true },
            columns: [
                //do not forget about column ids
                {dataIndex: 'company',      id:'company',       header:'Company'},
                {dataIndex: 'price',        id:'price',         header:'Price'},
                {dataIndex: 'change',       id:'change',        header:'Change'},
                {dataIndex: 'pctChange',    id:'pctChange',     header:'% Change'},
                {dataIndex: 'lastChange',   id:'lastChange',    header:'Last Updated'}
            ]
        }),
        stateful: true //enable rememebering state
    });
</script>
<!-- ON PAGE LOAD -->
<script type="text/javascript">
Ext.onReady(function () {
    Ext.QuickTips.init();

    new Ext.Viewport({
        layout: 'border',
        items: [grid]
    });
});
</script>
</head>
<body></body>
</html>
```
