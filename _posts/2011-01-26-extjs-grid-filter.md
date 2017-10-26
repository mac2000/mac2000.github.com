---
layout: post
title: ExtJs grid filter

tags: [datagrid, ext, extjs, filter, grid, gridpanel, messagebox, qtip, quicktip, renderer, tooltip]
---

Example of simple grid with filters

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>SenchaGridFilter</title>
<link href="ext-3.3.1/resources/css/ext-all.css" rel="stylesheet" type="text/css" />
<!-- styles for filters -->
<link rel="stylesheet" type="text/css" href="ext-3.3.1/examples/ux/gridfilters/css/GridFilters.css" />
<link rel="stylesheet" type="text/css" href="ext-3.3.1/examples/ux/gridfilters/css/RangeMenu.css" />

<script type="text/javascript" src="ext-3.3.1/adapter/ext/ext-base.js" ></script>
<script type="text/javascript" src="ext-3.3.1/ext-all.js" ></script>
<!-- scripts for filters -->
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/menu/RangeMenu.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/menu/ListMenu.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/GridFilters.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/filter/Filter.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/filter/StringFilter.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/filter/DateFilter.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/filter/ListFilter.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/filter/NumericFilter.js"></script>
<script type="text/javascript" src="ext-3.3.1/examples/ux/gridfilters/filter/BooleanFilter.js"></script>

<!-- TMP HELPER FUNCS -->
<script type="text/javascript">
function getRandomInt(min, max)
{
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
</script>

<!-- LOCAL DATA -->
<script type="text/javascript">
var mySeg = []; //<-- this is some values for ENUM
mySeg.push('a');
mySeg.push('b');
mySeg.push('c');
mySeg.push('aa');
mySeg.push('bb');
mySeg.push('cc');

var myData = []; //<-- this is my test local storage
for(var i = 1; i <= 30; i++) {
    var row = [];//<-- row

    var d = getRandomInt(1, 30);
    var m = getRandomInt(1, 12);
    var y = 2011;
    var h = getRandomInt(1, 23);
    var mm = getRandomInt(1, 59);
    var s = getRandomInt(1, 59);
    var dd = y + '/' + m + '/' + d + ' 00:00:00';

    row.push(dd);//<-- col1: data

    row.push(getRandomInt(1, 10000));//<-- col2: id
    row.push('Company'+i);//<-- col3: company

    if(getRandomInt(1, 100) % 2 == 0) {//<-- col4: notepads
        row.push(getRandomInt(1, 3));
    } else {
        row.push(1);
    }

    if(getRandomInt(1, 100) % 2 == 0) {//<-- col5: vacancies
        row.push(getRandomInt(1, 100));
    } else {
        row.push(0);
    }

    row.push((getRandomInt(1, 100) % 2 == 0));//<-- col6: reg

    row.push(mySeg[getRandomInt(0, mySeg.length - 1)]);//<-- col7: seg enum

    var hdn1 = getRandomInt(1, 10);
    var hdn2 = getRandomInt(1, 10);

    row.push(hdn1+hdn2);//<-- col8: sum of two fields
    row.push(hdn1);//<-- col9: hdn1
    row.push(hdn2);//<-- col10: hdn2

    myData.push(row);
}
</script>

<!-- STORE -->
<script type="text/javascript">
var store = new Ext.data.ArrayStore({//<-- declare local storege and its fields
    fields: [
        {name: 'lastdate', type: 'date'},
        {name: 'id', type: 'int'},
        {name: 'company'},
        {name: 'notepads', type: 'int'},
        {name: 'vacancies', type: 'int'},
        {name: 'reg', type: 'bool'},
        {name: 'seg'},
        {name: 'cust', type: 'int'},
        {name: 'hdn1', type: 'int'},
        {name: 'hdn2', type: 'int'}
    ]
});
store.loadData(myData);
</script>

<!-- RENDERERS -->
<script type="text/javascript">
function reg(val) {
    if (val == true) {
        return '<span style="color:green;">Y</span>';
    } else {
        return '<span style="color:red;">N</span>';
    }
}
function vacancies(val) {
    if (val == 0) {
        return '<span style="color:red;">0</span>';
    }
    return val;
}
function notepads(val) {
    if (val > 1) {
        return '<span style="color:red;">'+val+'</span>';
    }
    return val;
}
function seg(val) {
    if (val == 'a' || val == 'aa') {
        return '<span style="color:green;">'+val+'</span>';
    }
    else if (val == 'b' || val == 'bb') {
        return '<span style="color:blue;">'+val+'</span>';
    }
    else if (val == 'c' || val == 'cc') {
        return '<span style="color:red;">'+val+'</span>';
    }
    return val;
}
function cust(val, meta, record) {//<-- here is tooltip and messagebox
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
</script>

<!-- FILTERS -->
<script type="text/javascript">
var filters = new Ext.ux.grid.GridFilters({//<-- declaring filters
    encode: false,//encode, // json encode the filter query
    local: true,//local,   // defaults to false (remote filtering)
    filters: [
        {type:'date', dataIndex:'lastdate'},
        {type:'numeric', dataIndex:'id'},
        {type:'string', dataIndex:'company'},
        {type:'numeric', dataIndex:'notepads'},
        {type:'numeric', dataIndex:'vacancies'},
        {type:'boolean', dataIndex:'reg'},
        {type:'list', dataIndex:'seg', options: mySeg},//<-- mySeg - my local array of strings
        {type:'numeric', dataIndex:'cust'}
    ]
});
</script>

<!-- GRID -->
<script type="text/javascript">
var grid = new Ext.grid.GridPanel({//<-- standart grid
    region: 'center',
    store: store,
    loadMask: true,
    plugins: [filters],//<-- here is our filter
    colModel: new Ext.grid.ColumnModel({
        defaults: { sortable: true },
        columns: [
            {header:'Date', dataIndex:'lastdate', renderer: Ext.util.Format.dateRenderer('d/m/Y')},
            {header:'ID', dataIndex:'id'},
            {header:'Company', dataIndex:'company'},
            {header:'Notepads', dataIndex:'notepads', renderer : notepads},
            {header:'Vacancies', dataIndex:'vacancies', renderer : vacancies},
            {header:'Reg', dataIndex:'reg', renderer : reg},
            {header:'Seg', dataIndex:'seg', renderer : seg},
            {header:'Cust', dataIndex:'cust', renderer: cust}
        ]
    }),
    viewConfig: { forceFit: true }
});
</script>

<!-- ON LOAD -->
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

<body>
</body>
</html>
```
