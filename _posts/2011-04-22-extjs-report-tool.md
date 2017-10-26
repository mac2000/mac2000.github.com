---
layout: post
title: ExtJs Report tool

tags: [TODO]
---

Sample maded for rabota.ua sales department.

Here is how it looks like:

![](/images/wp/110.png)

Main page code:

```csharp
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Default.aspx.cs" Inherits="RabotaUA.Sales._Default" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head id="Head1" runat="server">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>TEST2</title>

    <style type="text/css">
        #loading{
            height:auto;
            position:absolute;
            left:45%;
            top:40%;
            padding:2px;
            z-index:20001;
        }
        #loading a {
            color:#225588;
        }
        #loading .loading-indicator{
            background:white;
            color:#444;
            font:bold 13px Helvetica, Arial, sans-serif;
            height:auto;
            margin:0;
            padding:10px;
        }
        #loading-msg {
            font-size: 10px;
            font-weight: normal;
        }

        .notebooklinks {border-collapse:collapse;margin:auto;margin-top:10px;}
        .notebooklinks th, .notebooklinks td {border:1px solid #999;padding:5px;}

        .rating_holder, .rating_holder tr, .rating_holder td, .rating_item {height:10px;line-height:10px;overflow:hidden;}
        .rating_holder, .rating_holder tr {width:50px;}
        .rating_holder td, .rating_item {width:10px;}
        .rating_item {text-indent:-9000px;display:block;background:transparent url(http://rabota.ua/Theme/img/star_small.gif) no-repeat 0 0;}
        .rating_item_starred {background-position:0 -20px;}

        #nav .x-btn-mc {text-align:left;}
        .header-icon {height:16px;line-height:16px;padding-left:18px;background-repeat:no-repeat;background-position:0 50%;}
        .cell-icon {height:16px;width:16px;line-height:16px;background-repeat:no-repeat;background-position:50% 50%;}
        .x-grid-panel td div {line-height:16px;}

        .rua_icon {background-image:url(http://rabota.ua/images/icons/rabota.gif);}
        .hh_icon {background-image:url(http://rabota.ua/images/icons/hh.gif);}
        .work_icon {background-image:url(http://rabota.ua/images/icons/work.gif);}

        .y {background:#ffeb9c;color:#9c6500;}
        .y a {color:#9c6500!important;}
        .g {background:#c6efce;color:#006100;}
        .g a {color:#006100!important;}
        .r {background:#ffc7ce;color:#9c0006;}
        .r a {color:#9c0006!important;}
    </style>

    <script type="text/javascript">
        function m(msg) {
            document.getElementById('loading-msg').innerHTML = msg;
        }
    </script>

</head>
<body>
<div id="loading-mask" style=""></div>
<div id="loading">
    <div class="loading-indicator"><img src="<%= ResolveUrl("~/ext/examples/shared/extjs/images/extanim32.gif")%>" width="32" height="32" style="margin-right:8px;float:left;vertical-align:top;"/>Report Tool - <a href="http://rabota.ua">rabota.ua</a><br /><span id="loading-msg">Загрузка рисунков и стилей...</span></div>
</div>

<link href="<%= ResolveUrl("~/ext/resources/css/ext-all.css")%>" rel="stylesheet" type="text/css" />
<link href="<%= ResolveUrl("~/ext/examples/ux/css/ux-all.css")%>" rel="stylesheet" type="text/css" />
<script type="text/javascript">    m('Закгрузка базовых элементов...');</script>
<script type="text/javascript" src="<%= ResolveUrl("~/ext/adapter/ext/ext-base.js")%>"></script>
<script type="text/javascript">    m('Загрузка компонентов интерфейсов...');</script>
<script type="text/javascript" src="<%= ResolveUrl("~/ext/ext-all-debug.js")%>"></script>
<script type="text/javascript">    m('Загрузка дополнительных компонентов...');</script>
<script type="text/javascript" src="<%= ResolveUrl("~/ext/examples/ux/ux-all-debug.js")%>"></script>
<script type="text/javascript">    m('Инициализация...');</script>
<script type="text/javascript" src="<%= ResolveUrl("~/TDGi.iconMgr/TDGi.iconMgr/TDGi.iconMgr.js")%>"></script>
<script type="text/javascript" src="<%= ResolveUrl("~/ext/examples/shared/extjs/App.js")%>"></script>
<script type="text/javascript" src="<%= ResolveUrl("~/Scripts/App/init.js")%>"></script>
<script type="text/javascript" src="<%= ResolveUrl("~/Scripts/App/renderers.js")%>"></script>
<%-- REPORTS --%>
<script type="text/javascript" src="<%= ResolveUrl("~/Scripts/App/reports/AttractiveProgram.js")%>"></script>
<script type="text/javascript" src="<%= ResolveUrl("~/Scripts/App/reports/ResumeBaseUse.js")%>"></script>
<script type="text/javascript" src="<%= ResolveUrl("~/Scripts/App/reports/AttractedCompany.js")%>"></script>
<script type="text/javascript" src="<%= ResolveUrl("~/Scripts/App/reports/TrophyBrandCompany.js")%>"></script>
<script type="text/javascript" src="<%= ResolveUrl("~/Scripts/App/reports/CVBaseAccess.js")%>"></script>

<form id="form1" runat="server"></form></body></html>
```

Code is copied from extjs examples, first of all we showing loader, and loading all needed files.

Note: **TDGi.iconMgr** - cool extjs plugin that adds famfam icons to your project.

After all files loaded, we load our init.js, and reports (each report has its own js file)

So here is init.js

```js
var _DEFAULT_PERPAGE = 20;

/* region Logger functions */
var log_item_tpl = new Ext.Template('<table cellpadding="0" cellspacing="0" border="0" width="100%"><tr>'
        + '<td valign="top" align="left" width="10" style="white-space:nowrap;padding-right:5px;">{date}</td>'
        + '<td valign="top" align="left">{msg}</td>' + '</tr></table>');

var store_exception_log_row_tpl = new Ext.Template('<tr><td valign="top" align="left" width="10" style="white-space:nowrap;padding:2px;border:1px solid #999;">{key}</td><td valign="top" align="left" style="padding:2px;border:1px solid #999;">{val}</td></tr>');
var store_exception_log_tpl = new Ext.Template('<div><a href="#additional_info" onclick="this.parentNode.getElementsByTagName(\'DIV\').item(0).style.display=\'block\';this.style.display=\'none\';return false;">Показать дополнительную информацию</a><div style="display:none"><table cellpadding="0" cellspacing="0" border="0" width="100%" style="border-collapse:collapse;">{rows}</table></div></div>');

function le(msg) {
    return logmessage('error', msg);
}

function li(msg) {
    return logmessage('info', msg);
}

function l(msg) {
    return logmessage('warning', msg);
}

function logmessage(type, msg) {
    var h = Ext.get('log').query('.x-panel-body');
    if (h.length == 0) {
        Ext.MessageBox.alert('Ошибка',
                'Похоже что страница не загрузилась, попробуйте обновить ее.');
        return false;
    }
    h = h[0];

    var d = new Date();

    switch (type) {
        case 'error' :
            var color = '#900';
            break;
        case 'notice' :
        case 'info' :
            var color = '#090';
            break;
        default :
            var color = '#000';
    }

    h.innerHTML += log_item_tpl.apply({
                date : d.toLocaleString(),
                msg : '<span style="color:' + color + '">' + msg + '</span>'
            });
    return true;
}
/* endregion */

var App = new Ext.App();

/* region Loggable Store */
var LoggableStoreBase = Ext.extend(Ext.data.JsonStore, {
    store_name : 'справочник',
    constructor : function() {
        LoggableStoreBase.superclass.constructor.apply(this, arguments);

        // TODO: IMPLEMENT OTHER EVENTS
        // http://dev.sencha.com/deploy/dev/docs/?class=Ext.data.JsonStore
        this.on('beforeload', function() {
                    Ext.get(document.body).mask('Загрузка данных...');
                });
        this.on('write', function(store, action, result, res, rs) {
                    Ext.get(document.body).unmask();
                    li(this.store_name + ' ' + action);
                    App.setAlert('', this.store_name + ' - транзакция '
                                    + action + ' успешно завершена');
                });
        this.on('load', function(res) {
                    Ext.get(document.body).unmask();
                    li(this.store_name + ' loaded');
                    App.setAlert('', this.store_name + ' - загрузка завершена');
                });
        this.on('exception', function(proxy, type, action, options, res) {
                    var msg = 'ошбика';
                    if (type === 'remote') {
                        msg = res.message;
                    } else if (type === 'response') {
                        var msg = '';
                        if (res.isTimeout) {
                            msg = 'Истекло время ожидания.';
                        }
                        if (res.status != 200) {
                            msg = res.statusText;
                        } else {
                            msg = options.reader.jsonData.message;
                        }
                    }

                    var rows = [];
                    rows.push(store_exception_log_row_tpl.apply({
                                key : 'type',
                                val : type
                            }));
                    rows.push(store_exception_log_row_tpl.apply({
                                key : 'action',
                                val : action
                            }));
                    rows.push(store_exception_log_row_tpl.apply({
                                key : 'status',
                                val : res.status
                            }));
                    rows.push(store_exception_log_row_tpl.apply({
                                key : 'statusText',
                                val : res.statusText
                            }));
                    rows.push(store_exception_log_row_tpl.apply({
                                key : 'isTimeout',
                                val : res.isTimeout ? 'true' : 'false'
                            }));
                    rows.push(store_exception_log_row_tpl.apply({
                                key : 'url',
                                val : options.url
                            }));

                    var info = store_exception_log_tpl.apply({
                                rows : rows.join('')
                            })

                    msg = '<b>' + this.store_name + '</b><br /><br />' + msg
                            + '<br /><br />' + info + '';

                    Ext.Msg.show({
                                title : 'ОШИБКА',
                                msg : msg,
                                icon : Ext.MessageBox.ERROR,
                                buttons : Ext.Msg.OK,
                                fn : function() {
                                    location = location;
                                }
                            });
                });
    }
});
Ext.reg("LoggableStoreBase", LoggableStoreBase);
/* endregion */

/* region Dictionary Store Base */
var DictionaryBase = Ext.extend(LoggableStoreBase, {
            constructor : function() {
                Ext.apply(this, {
                            remoteSort : true,
                            autoSave : false,
                            batch : false,
                            autoLoad : true
                        });
                DictionaryBase.superclass.constructor.apply(this, arguments);
            }
        });
Ext.reg("DictionaryBase", DictionaryBase);

DictionaryBaseFactory = function(store_name, json_url) {
    var id = 'dict'
            + json_url.replace(new RegExp('[^a-z0-9]+', 'gi'), '_')
                    .toLowerCase();
    return new DictionaryBase({
                url : json_url,
                store_name : store_name,
                id : id
            });
}
/* endregion */

/* region Report Store Base */
var ReportBase = Ext.extend(LoggableStoreBase, {
            constructor : function() {
                Ext.apply(this, {
                            reader : new Ext.data.JsonReader(),
                            autoDestroy: true,
                            sortInfo : {
                                field : 'NotebookID',
                                direction : 'ASC'
                            },
                            writer : new Ext.data.JsonWriter({
                                        encode : true,
                                        writeAllFields : true
                                    }),
                            autoSave : true,
                            batch : false,
                            remoteSort : true,
                            autoLoad : {
                                params : {
                                    start : 0,
                                    limit : _DEFAULT_PERPAGE
                                }
                            }
                        });
                ReportBase.superclass.constructor.apply(this, arguments);
            }
        });
Ext.reg("ReportBase", ReportBase);
/* endregion */

/* region Report Grid Base */
var ReportGridBase = Ext.extend(Ext.grid.GridPanel, {
    constructor : function() {
        Ext.apply(this, {
            region : 'center',
            viewConfig : {
                forceFit : true
            },
            bbar : new Ext.PagingToolbar({
                pageSize : _DEFAULT_PERPAGE,
                displayInfo : true,
                displayMsg : 'Отображены записи {0} - {1} из {2}',
                emptyMsg : 'Нет записей',
                items : [new Ext.form.ComboBox({
                    name : 'perpage',
                    width : 40,
                    store : new Ext.data.ArrayStore({
                                fields : ['id'],
                                data : [['10'], ['20'], ['30'], ['40'], ['50'],
                                        ['60'], ['70'], ['80'], ['90'], ['100']]
                            }),
                    mode : 'local',
                    value : _DEFAULT_PERPAGE,
                    listWidth : 40,
                    triggerAction : 'all',
                    displayField : 'id',
                    valueField : 'id',
                    editable : false,
                    forceSelection : true,
                    listeners : {
                        'select' : function(dropdown, record, index) {
                            var bbar = dropdown
                                    .findParentByType(Ext.PagingToolbar);
                            if (bbar) {
                                bbar.pageSize = parseInt(record.get('id'), 10);
                                bbar.doLoad(bbar.cursor);
                            }
                        }
                    }
                }), '-', {
                    xtype : 'button',
                    iconCls : Ext.ux.TDGi.iconMgr.getIcon('bin_closed'),
                    text : 'Сбросить фильтры',
                    handler : function(btn, e) {
                        var grid = btn.findParentByType(Ext.grid.GridPanel);
                        if (grid && grid.filters && grid.filters.clearFilters)
                            grid.filters.clearFilters();
                    }
                }, {
                    xtype : 'button',
                    iconCls : Ext.ux.TDGi.iconMgr.getIcon('page_white_excel'),
                    text : 'Экспорт в Excel',
                    handler : function(btn, e) {
                        var grid = btn.findParentByType(Ext.grid.GridPanel);
                        var url = grid.store.proxy.buildUrl('read')
                                + '?excel=1&sort=' + grid.store.sortInfo.field
                                + '&dir=' + grid.store.sortInfo.direction;
                        var filter = '&filter='
                                + escape(grid.filters.buildQuery(grid.filters
                                        .getFilterData()).filter);
                        if (filter != '&filter=undefined')
                            url += filter;
                        location = url;
                    }
                }]
            }),
            listeners : {
                'beforerender' : {
                    fn : function() {
                        this.getBottomToolbar().bindStore(this.store);
                    }
                }
            }
        });
        ReportGridBase.superclass.constructor.apply(this, arguments);
    }
});
Ext.reg("ReportGridBase", ReportGridBase);
/* endregion */

var DICTS = {};
var PANELS = {};
var REPORT_BUTTONS = [];

Ext.onReady(function() {
(function   () {
                Ext.QuickTips.init();

                /* region Dictionaries */
                DICTS.BRANCHES = DictionaryBaseFactory('Отрасли',
                        '/json/Dictionaries/Branches.ashx');
                DICTS.CITIES = DictionaryBaseFactory('Регионы',
                        '/json/Dictionaries/Cities.ashx');
                DICTS.MANAGERS = DictionaryBaseFactory('Менеджеры',
                        '/json/Dictionaries/Managers.ashx');
                DICTS.NOTEBOOK_STATUSES = DictionaryBaseFactory(
                        'Статусы блокнота',
                        '/json/Dictionaries/NotebookStatuses.ashx');
                /* endregion */

                PANELS.NAV = new Ext.Panel({
                            id : 'nav',
                            region : 'west',
                            title : 'Навигация',
                            layout : 'accordion',
                            margins : '5 0 0 5',
                            collapseMode : 'mini',
                            split : true,
                            collapsible : true,
                            collapsed : true,
                            width : 200,
                            items : [{
                                        title : 'Отчеты',
                                        padding : 5,
                                        layout : 'vbox',
                                        align : 'stretch',
                                        defaults : {
                                            margins : '0 0 5 0',
                                            width : '100%'
                                        },
                                        items : REPORT_BUTTONS
                                    }, {
                                        title : 'Помощь',
                                        padding : 5,
                                        autoLoad: { url: '/help.html'}
                                    }, {
                                        title : 'Утилиты',
                                        padding : 5,
                                        layout : 'vbox',
                                        align : 'stretch',
                                        defaults : {
                                            margins : '0 0 5 0',
                                            width : '100%'
                                        },
                                        items : [{
                                            xtype : 'button',
                                            text : 'Иконки',
                                            iconCls : Ext.ux.TDGi.iconMgr
                                                    .getIcon('application_view_icons'),
                                            handler : function() {
                                                Ext.ux.TDGi.iconBrowser.show();
                                            }
                                        }]
                                    }]
                        });

                PANELS.LOG = new Ext.Panel({
                            region : 'south',
                            title : 'Журнал событий',
                            collapseMode : 'mini',
                            id : 'log',
                            autoScroll : true,
                            height : 100,
                            padding : 5,
                            collapsible : true,
                            collapsed : true,
                            margins : '0 5 5 5',
                            split : true
                        });

                PANELS.TABS = new Ext.TabPanel({
                            region : 'center',
                            margins : '5 5 0 0',
                            plugins : new Ext.ux.TabCloseMenu(),
                            activeTab : 0,
                            items : [{
                                        title : 'Главная',
                                        closable : false,
                                        padding : 5,
                                        autoLoad: { url: '/hometab.html'}
                                    }]
                        });

                new Ext.Viewport({
                            id : 'viewport',
                            layout : 'border',
                            items : [PANELS.TABS, PANELS.NAV, PANELS.LOG]
                        });

                hideMask.defer(250);

            }).defer(500);

            var hideMask = function() {
                Ext.get('loading').remove();
                Ext.fly('loading-mask').fadeOut({
                            remove : true
                        });
            };

        });

var maskingAjax = new Ext.data.Connection({
            listeners : {
                'beforerequest' : {
                    fn : function(con, opt) {
                        Ext.get(document.body).mask('Загрузка...');
                    },
                    scope : this
                },
                'requestcomplete' : {
                    fn : function(con, res, opt) {
                        Ext.get(document.body).unmask();
                    },
                    scope : this
                },
                'requestexception' : {
                    fn : function(con, res, opt) {
                        Ext.get(document.body).unmask();
                    },
                    scope : this
                }
            }
        });

function getNotebookLinks(id, source) {
    maskingAjax.request({
        url : '/json/AttractiveProgram/GetInfo.ashx',
        failure : function() {
            Ext.Msg.show({
                        title : 'ОШИБКА',
                        msg : 'Произошла ошибка при передаче данных',
                        icon : Ext.MessageBox.ERROR,
                        buttons : Ext.Msg.OK
                    });
        },
        success : function(r, o) {
            var resp = Ext.decode(r.responseText);
            if (!resp.success) {
                Ext.Msg.show({
                            title : 'ОШИБКА',
                            msg : resp.message,
                            icon : Ext.MessageBox.ERROR,
                            buttons : Ext.Msg.OK
                        });
            } else {
                var notebooks = resp.data;
                var site = (source == 1) ? 'work.ua' : 'hh.ua';
                var msg = 'У компании несколько блокнотов на сайте ' + site;
                msg += '<div style="height:200px;overflow-y:scroll;"><table class="notebooklinks" cellspacing="0" cellpadding="0" border="0"><tr><th>Блокнот</th><th>Количество вакансий</th></tr>';

                for (var i = 0; i < notebooks.length; i++) {
                    var notebook = notebooks[i];

                    var link = '';
                    if (source == 1) {
                        link = '<a href="http://www.work.ua/jobs/by-company/'
                                + notebooks[i].CompanyId
                                + '/" target="_blank">'
                                + notebooks[i].CompanyId + '</a>';
                    } else if (source == 2) {
                        link = '<a href="http://hh.ua/employer/'
                                + notebooks[i].CompanyId + '" target="_blank">'
                                + notebooks[i].CompanyId + '</a>';
                    }

                    msg += '<tr><td>' + link + '</td><td>'
                            + notebooks[i].VacancyCount + '</td></tr>';
                }

                msg += '</table></div>';

                Ext.Msg.show({
                            title : 'Ссылки',
                            msg : msg,
                            buttons : Ext.Msg.OK
                        });
            }
        },
        params : {
            id : id,
            source : source
        }
    });
}
```

Have look at extended stores and grid this is pretty standard way in extjs to extend base classes.

After all defined we create viewport and fill it with some UI.

Here is example of report:

```js
REPORT_BUTTONS.push({
    xtype : 'button',
    iconCls : Ext.ux.TDGi.iconMgr.getIcon('report'),
    text : 'Trophy Brand Company',
    handler : function() {
        PANELS.TABS.add({
            title : 'Trophy Brand Company',
            closable : true,
            iconCls : Ext.ux.TDGi.iconMgr.getIcon('report'),
            layout : 'border',
            items : [new ReportGridBase({
                store : new ReportBase({
                    store_name : 'Trophy Brand Company',
                    proxy : new Ext.data.HttpProxy({
                                api : {
                                    create : 'NOT_IMPLEMENTED',
                                    read : '/json/TrophyBrandCompany/Report.ashx',
                                    update : 'NOT_IMPLEMENTED',
                                    destroy : 'NOT_IMPLEMENTED'
                                }
                            })
                }),
                plugins : [new Ext.ux.grid.GridFilters({
                            encode : true,
                            local : false,
                            filters : [{
                                        type : 'boolean',
                                        dataIndex : 'IsPaidRabota'
                                    }, {
                                        type : 'boolean',
                                        dataIndex : 'IsActiveCompany'
                                    }, {
                                        type : 'date',
                                        dateFormat : 'd/m/Y',
                                        dataIndex : 'MaxExpiryDate'
                                    }, {
                                        type : 'numeric',
                                        dataIndex : 'NotebookID',
                                        menuItems : ['eq']
                                    }, {
                                        type : 'list',
                                        dataIndex : 'BranchID',
                                        store : DICTS.BRANCHES,
                                        labelField : 'Name'
                                    }, {
                                        type : 'list',
                                        dataIndex : 'CityID',
                                        store : DICTS.CITIES,
                                        labelField : 'Name'
                                    }, {
                                        type : 'numeric',
                                        dataIndex : 'VacancyCount'
                                    }, {
                                        type : 'numeric',
                                        dataIndex : 'WorkVacancyCount'
                                    }, {
                                        type : 'numeric',
                                        dataIndex : 'HHVacancyCount'
                                    }, {
                                        type : 'numeric',
                                        dataIndex : 'MaxWorkVacancyCount'
                                    }, {
                                        type : 'boolean',
                                        dataIndex : 'IsPaidWork'
                                    }, {
                                        type : 'boolean',
                                        dataIndex : 'IsPaidHH'
                                    }, {
                                        type : 'list',
                                        dataIndex : 'SegmentCategory',
                                        options : [[1, '1 - AA'], [2, '2 - AB']]
                                    }, {
                                        type : 'list',
                                        dataIndex : 'ManagerID',
                                        store : DICTS.MANAGERS,
                                        labelField : 'Name'
                                    }, {
                                        type : 'numeric',
                                        dataIndex : 'OpenContactCount'
                                    }, {
                                        type : 'numeric',
                                        dataIndex : 'ResumeViewCount'
                                    }, {
                                        type : 'boolean',
                                        dataIndex : 'IsPaidCVBaseAccess'
                                    }]
                        })],
                colModel : new Ext.grid.ColumnModel({
                    defaults : {
                        sortable : true
                    },
                    columns : [{
                        dataIndex : 'IsActiveCompany',
                        header : 'Акт.',
                        tooltip : 'Активная компания',
                        renderer : function(val, meta, record) {
                            var icon = Ext.ux.TDGi.iconMgr.getIcon('star_gey');
                            if (val) {
                                icon = Ext.ux.TDGi.iconMgr.getIcon('star');
                            }

                            return '<div class="cell-icon ' + icon + '"></div>';
                        }
                    }, {
                        dataIndex : 'MaxExpiryDate',
                        header : 'Сервис активирован до',
                        tooltip : 'Сервис активирован до',
                        renderer : RENDERERS.NullableDateRenderer
                    }, {
                        dataIndex : 'NotebookID',
                        header : 'ID',
                        tooltip : 'ID блокнота на rabota.ua',
                        renderer : RENDERERS.NotebookID
                    }, {
                        dataIndex : 'Name',
                        header : 'Название',
                        tooltip : 'Название',
                        tooltip : 'Название компании на rabota.ua'
                    }, {
                        dataIndex : 'BranchID',
                        header : 'Отрасль',
                        tooltip : 'Отрасль блокнота на rabota.ua',
                        renderer : RENDERERS.BranchID
                    }, {
                        dataIndex : 'TicketCount',
                        header : '<div class="header-icon '
                                + Ext.ux.TDGi.iconMgr.getIcon('control_pause')
                                + '">Платных вакансий на счету</div>',
                        tooltip : 'Платных вакансий на счету на rabota.ua',
                        align : 'right',
                        renderer : RENDERERS.TicketCount
                    }, {
                        dataIndex : 'PaidVacancyCount',
                        header : '<div class="header-icon '
                                + Ext.ux.TDGi.iconMgr
                                        .getIcon('control_play_blue')
                                + '">Платных вакансий опубликовано</div>',
                        tooltip : 'Платных вакансий опубликовано на rabota.ua',
                        align : 'right',
                        renderer : RENDERERS.PaidVacancyCount
                    }, {
                        dataIndex : 'VacancyCount',
                        header : '<div class="header-icon rua_icon"></div>',
                        tooltip : 'Количество вакансий на rabota.ua',
                        align : 'right',
                        renderer : RENDERERS.VacancyCount
                    }, {
                        dataIndex : 'HHVacancyCount',
                        header : '<div class="header-icon hh_icon"></div>',
                        tooltip : 'Количество вакансий на hh.ua',
                        align : 'right',
                        renderer : RENDERERS.HHVacancyCount
                    }, {
                        dataIndex : 'MaxWorkVacancyCount',
                        header : '<div class="header-icon work_icon">Max</div>',
                        tooltip : 'Максимальное количество вакансий в блокноте на work.ua',
                        align : 'right',
                        renderer : RENDERERS.MaxWorkVacancyCount
                    }, {
                        dataIndex : 'IsPaidRabota',
                        header : '<div class="header-icon rua_icon">VIP</div>',
                        tooltip : 'Платят нам',
                        renderer : RENDERERS.IsPaidRabota
                    }, {
                        dataIndex : 'IsPaidWork',
                        header : '<div class="header-icon work_icon">VIP</div>',
                        tooltip : 'Платит ли компания work\'у',
                        renderer : RENDERERS.IsPaidWork
                    }, {
                        dataIndex : 'IsPaidHH',
                        header : '<div class="header-icon hh_icon">VIP</div>',
                        tooltip : 'Платит ли компания hh',
                        renderer : RENDERERS.IsPaidHH
                    }, {
                        dataIndex : 'WorkVacancyCount',
                        header : '<div class="header-icon work_icon">Max</div>',
                        tooltip : 'Количество вакансий на work.ua',
                        align : 'right',
                        renderer : RENDERERS.WorkVacancyCount
                    }, {
                        dataIndex : 'SegmentCategory',
                        header : 'Сегмент',
                        tooltip : 'Сегмент компании на rabota.ua'
                    }, {
                        dataIndex : 'ManagerID',
                        header : 'Менеджер',
                        tooltip : 'Менеджер за которым закреплен блокнот',
                        renderer : RENDERERS.ManagerID
                    }, {
                        dataIndex : 'CityID',
                        header : 'Регион',
                        tooltip : 'Регион блокнота на rabota.ua',
                        renderer : RENDERERS.CityID
                    }, {
                        dataIndex : 'LastPaidDate',
                        header : 'Посл. пропл.',
                        tooltip : 'Дата последней проплаты',
                        renderer : RENDERERS.NullableDateRenderer
                    }, {
                        dataIndex : 'OpenContactCount',
                        header : 'Контакты базы',
                        tooltip : 'Открыто контактов резюме',
                        align : 'right'
                    }, {
                        dataIndex : 'ResumeViewCount',
                        header : 'Просмотры базы',
                        tooltip : 'Просмотров базы резюме',
                        align : 'right'
                    }, {
                        dataIndex : 'IsPaidCVBaseAccess',
                        header : 'Платный доступ к CVDB',
                        tooltip : 'Активирован ли платный доступ к CV DB?',
                        renderer: RENDERERS.IsPaidCVBaseAccess
                    }, {
                        dataIndex : 'PaidAccessExpiryDate',
                        header : 'Проплачено до',
                        tooltip : 'Платный доступ к базе резюме проплачен до',
                        renderer : RENDERERS.NullableDateRenderer
                    }, {
                        dataIndex : 'Rating',
                        header : 'Привлекательность',
                        tooltip : 'Привлекательность компании',
                        renderer : RENDERERS.Rating
                    }]
                })
            })]
        }).show();
    }
});
```

Because i have extended grid and store, all i need is to declare grid columns and filters.

Many of fields have own renderers here is their code:

```js
var RENDERERS = {};

RENDERERS.NotebookID = function(val, meta, record) {
    return '<a href="http://rabota.ua/company' + val
            + '" title="Перейти на страницу компании" target="_blank">' + val
            + '</a>';
}

RENDERERS.CityID = function(val, meta, record) {
    var rec = DICTS.CITIES.getById(val);
    if (!rec) {
        meta.attr = 'ext:qtip="Похоже что справочник регионов не был загружен, необходимо обновить отчет либо перезагрузить страницу"';
        return '<span style="color:#999">н.д.</span>';
    }
    meta.attr = 'ext:qtip="' + rec.get('Name') + '"';
    return rec.get('Name');
}

RENDERERS.BranchID = function(val, meta, record) {
    var rec = DICTS.BRANCHES.getById(val);
    if (!rec) {
        meta.attr = 'ext:qtip="Похоже что справочник отраслей не был загружен, необходимо обновить отчет либо перезагрузить страницу"';
        return '<span style="color:#999">н.д.</span>';
    }
    meta.attr = 'ext:qtip="' + rec.get('Name') + '"';
    return rec.get('Name');
}

RENDERERS.ManagerID = function(val, meta, record) {
    var rec = DICTS.MANAGERS.getById(val);
    if (!rec) {
        meta.attr = 'ext:qtip="Похоже что справочник менеджеров не был загружен, необходимо обновить отчет либо перезагрузить страницу"';
        return '<span style="color:#999">н.д.</span>';
    }
    var full_name = rec.get('Name');
    var short_name = '';

    var parts = full_name.split(' ');
    short_name = parts[0];
    if (parts.length > 1)
        short_name += ' ' + parts[1].substring(0, 1) + '.';

    meta.attr = 'ext:qtip="' + full_name + '"';
    return short_name;
}

RENDERERS.NotebookStateID = function(val, meta, record) {
    var rec = DICTS.NOTEBOOK_STATUSES.getById(val);
    if (!rec) {
        meta.attr = 'ext:qtip="Похоже что справочник статусов блокнота не был загружен, необходимо обновить отчет либо перезагрузить страницу"';
        return '<span style="color:#999">н.д.</span>';
    }
    var full_name = rec.get('Name');
    var short_name = '';

    var parts = full_name.split(' ');

    Ext.each(parts, function(item, index) {
                short_name += item.substring(0, 1);
            });

    meta.attr = 'ext:qtip="' + full_name + '"';

    return short_name;
}

RENDERERS.NullableDateRenderer = function(val, meta, record) {
    if (!val || val == null || val == '') {
        meta.attr = 'ext:qtip="Нет данных"';
        return '<span style="color:#999">н.д.</span>';
    }
    f = Ext.util.Format.dateRenderer('d/m/Y')
    var r = f(val);
    if (r == '01/01/0001') {
        meta.attr = 'ext:qtip="Нет данных"';
        return '<span style="color:#999">н.д.</span>';
    }

    return r;
}

RENDERERS.Rating = function(val, meta, record) {
    var r = '<table class="rating_holder" cellpadding="0" cellspacing="0" border="0" align="center"><tr>';
    for (var i = 0; i < val; i++) {
        r += '<td><span class="rating_item rating_item_starred">+</span></td>';
    }
    for (var i = val; i < 5; i++) {
        r += '<td><span class="rating_item">-</span></td>';
    }
    r += '</tr></table>';
    meta.attr = 'ext:qtip="' + val + ' звезд"';
    return r;
}

RENDERERS.WorkVacancyCount = function(val, meta, record) {
    if (record.get('IsMultiWork')) {
        var ruaid = record.get('NotebookID');
        meta.attr = 'ext:qtip="Суммарное количество вакансий в нескольких блокнотах"';

        return unescape('%u01A9')
                + ' <a href="javascript:void(0)" onclick="getNotebookLinks('
                + ruaid + ', 1);return false;">' + val + '</a>';
    }

    var id = record.get('WorkCompanyID');
    if (!id) {
        meta.attr = 'ext:qtip="Нет блокнота на work.ua"';
        return '<span style="color:#999">' + val + '</span>';
    }

    return '<a href="http://www.work.ua/jobs/by-company/' + id
            + '/" target="_blank">' + val + '</a>';
}

RENDERERS.HHVacancyCount = function(val, meta, record) {
    if (record.get('IsMultiHH')) {
        var ruaid = record.get('NotebookID');
        meta.attr = 'ext:qtip="Суммарное количество вакансий в нескольких блокнотах"';
        return unescape('%u01A9')
                + ' <a href="javascript:void(0)" onclick="getNotebookLinks('
                + ruaid + ', 2);return false;">' + val + '</a>';
    }

    var id = record.get('HHCompanyID');
    if (!id) {
        meta.attr = 'ext:qtip="Нет блокнота на hh.ua"';
        return '<span style="color:#999">' + val + '</span>';
    }
    return '<a href="http://hh.ua/employer/' + id + '" target="_blank">' + val
            + '</a>';
}

RENDERERS.IsPaidWork = function(val, meta, record) {
    var IsPaidRabota = record.get('IsPaidRabota');
    var icon = Ext.ux.TDGi.iconMgr.getIcon('bullet_toggle_minus');

    if (val) {
        if (typeof IsPaidRabota == 'undefined' || IsPaidRabota) {
            meta.attr = 'ext:qtip="Компания платит work\'у"';
            icon = Ext.ux.TDGi.iconMgr.getIcon('error');
        } else {
            meta.attr = 'ext:qtip="Компания платит work\'у и НЕ платит нам"';
            icon = Ext.ux.TDGi.iconMgr.getIcon('cancel');
        }
    }

    return '<div class="cell-icon ' + icon + '"></div>';
}

RENDERERS.IsPaidHH = function(val, meta, record) {
    var IsPaidRabota = record.get('IsPaidRabota');
    var icon = Ext.ux.TDGi.iconMgr.getIcon('bullet_toggle_minus');

    if (val) {
        if (typeof IsPaidRabota == 'undefined' || IsPaidRabota) {
            meta.attr = 'ext:qtip="Компания платит hh"';
            icon = Ext.ux.TDGi.iconMgr.getIcon('error');
        } else {
            meta.attr = 'ext:qtip="Компания платит hh и НЕ платит нам"';
            icon = Ext.ux.TDGi.iconMgr.getIcon('cancel');
        }
    }

    return '<div class="cell-icon ' + icon + '"></div>';
}

RENDERERS.IsPaidRabota = function(val, meta, record) {
    var icon = Ext.ux.TDGi.iconMgr.getIcon('bullet_toggle_minus');
    if (val) {
        icon = Ext.ux.TDGi.iconMgr.getIcon('coins');
    }

    return '<div class="cell-icon ' + icon + '"></div>';
}

RENDERERS.IsPaidCVBaseAccess = function(val, meta, record) {
    var icon = Ext.ux.TDGi.iconMgr.getIcon('bullet_toggle_minus');
    if (val) {
        icon = Ext.ux.TDGi.iconMgr.getIcon('coins');
    }

    return '<div class="cell-icon ' + icon + '"></div>';
}

RENDERERS.MaxWorkVacancyCount = function(val, meta, record) {
    var IsPaidRabota = record.get('IsPaidRabota');

    var cls = '';
    if (val >= 6) {
        meta.attr = 'ext:qtip="Эта компания платит work, за размещение вакансий<br />Больше 6 вакансий в одном блокноте"';
        cls = 'r';
    }

    if (IsPaidRabota && val >= 6) {
        meta.attr = 'ext:qtip="Эта компания платит work, за размещение вакансий<br />Больше 6 вакансий в одном блокноте<br />Но при этом платит нам"';
        cls = 'y';
    }

    if (cls)
        meta.css += ' ' + cls;
    return val;
}

RENDERERS.VacancyCount = function(val, meta, record) {
    var MaxWorkVacancyCount = record.get('MaxWorkVacancyCount');
    var HHVacancyCount = record.get('HHVacancyCount');
    var NotebookID = record.get('NotebookID');

    var cls = ''

    if (val > MaxWorkVacancyCount || val > HHVacancyCount) {
        if (val > MaxWorkVacancyCount)
            meta.attr = 'ext:qtip="У нас больше вакансий чем у work"';
        if (val > HHVacancyCount)
            meta.attr = 'ext:qtip="У нас больше вакансий чем у hh"';
        cls = 'g';
    }

    if (val == MaxWorkVacancyCount || val == HHVacancyCount) {
        if (val == MaxWorkVacancyCount)
            meta.attr = 'ext:qtip="У нас столько же вакансий как и у work"';
        if (val == HHVacancyCount)
            meta.attr = 'ext:qtip="У нас столько же вакансий как и у hh"';
        cls = 'y';
    }

    if (val < MaxWorkVacancyCount || val < HHVacancyCount) {
        if (val < MaxWorkVacancyCount)
            meta.attr = 'ext:qtip="У нас меньше вакансий чем у work"';
        if (val < HHVacancyCount)
            meta.attr = 'ext:qtip="У нас меньше вакансий чем у hh"';
        cls = 'r';
    }

    if (cls)
        meta.css += ' ' + cls;

    if (typeof NotebookID == 'undefined') {
        return val;
    } else {
        return '<a href="http://rabota.ua/company' + NotebookID
                + '" target="_blank">' + val + '</a>';
    }
}

RENDERERS.PaidVacancyCount = function(val, meta, record) {
    var VacancyCount = record.get('VacancyCount');
    if (typeof VacancyCount == 'undefined')
        return val;

    var cls = ''

    if (val < VacancyCount) {
        meta.attr = 'ext:qtip="Количество опубликованых платных вакансий меньше количества вакансий блокнота"';
        cls = 'y';
    }

    if (val == VacancyCount) {
        meta.attr = 'ext:qtip="Количество опубликованых платных вакансий равно количеству вакансий блокнота"';
        cls = 'g';
    }

    if (cls)
        meta.css += ' ' + cls;

    return val;
}

RENDERERS.TicketCount = function(val, meta, record) {
    var VacancyCount = record.get('VacancyCount');

    if (typeof VacancyCount == 'undefined')
        return val;

    var cls = ''

    if (val < VacancyCount) {
        meta.attr = 'ext:qtip="Количество билетиков меньше количества вакансий"';
        cls = 'y';
    }

    if (val > VacancyCount) {
        meta.attr = 'ext:qtip="Количество билетиков равно количеству вакансий"';
        cls = 'g';
    }

    if (cls)
        meta.css += ' ' + cls;

    return val;
}
```

Notice that in stores i use metadata driven json from asp handler that automaticaly generates all need stufs from returned DataTables.

In general handler looks like:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.IO;
using ExtJs;

namespace RabotaUA.Sales.json.TrophyBrandCompany
{
    /// <summary>
    /// Summary description for Report
    /// </summary>
    public class Report : IHttpHandler
    {

        public void ProcessRequest(HttpContext context)
        {
            context.Response.ContentType = "application/json";
            context.Response.Cache.SetCacheability(HttpCacheability.ServerAndPrivate);
            context.Response.Cache.SetExpires(DateTime.Now.AddDays(1));
            context.Response.Cache.SetMaxAge(new TimeSpan(1, 0, 0, 0));

            try
            {
                ExtJs.GridReader gr = new ExtJs.GridReader(Common.ConnectionString, "spAdmin3_TrophyBrandCompany_GetList", "NotebookID", context);

                if (context.Request.QueryString["excel"] == null)
                { context.Response.Write(gr); }
                else
                {
                    string file = gr.ToString();
                    try
                    {
                        File.Delete(file);
                    }
                    catch { }

                    context.Response.End();
                }
            }
            catch (Exception e)
            {

                ResponseBase resp = new ResponseBase(false, e.Message);
                context.Response.Write(resp);
            }
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```

All work done in ExtJS.GridReader and here it is:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Web;
using System.Data;
using System.Data.SqlClient;
using System.Text.RegularExpressions;
using RabotaUA.Model;
using System.IO;

namespace ExtJs
{
    public class GridReader
    {
        protected string connection_string = string.Empty;
        protected string procedure_name = string.Empty;
        protected string id_column_name = string.Empty;
        protected HttpContext context;
        protected DataSet ds;
        protected SqlDataAdapter da;
        protected SqlConnection con;
        protected SqlCommand cmd;

        protected int start = 0;
        protected int limit = 20;
        protected string sort = "";
        protected string dir = "ASC";

        public GridReader(string connection_string, string procedure_name, string id_column_name, HttpContext context)
        {
            this.connection_string = connection_string;
            this.procedure_name = procedure_name;
            this.id_column_name = id_column_name;
            this.sort = id_column_name;
            this.context = context;

            this.ds = new DataSet();
            this.con = new SqlConnection(this.connection_string);

            this.cmd = new SqlCommand(this.procedure_name, this.con);
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.CommandTimeout = 900;

            this.da = new SqlDataAdapter(cmd);

            if (!string.IsNullOrEmpty(context.Request.Params["start"])) start = int.Parse(context.Request.Params["start"]);
            if (!string.IsNullOrEmpty(context.Request.Params["limit"])) limit = int.Parse(context.Request.Params["limit"]);

            if (!string.IsNullOrEmpty(context.Request.Params["sort"])) sort = context.Request.Params["sort"];
            if (!string.IsNullOrEmpty(context.Request.Params["dir"])) dir = context.Request.Params["dir"];

            FilterParams fp = new FilterParams(context.Request.Params["filter"]);

            con.Open();
            System.Data.SqlClient.SqlCommandBuilder.DeriveParameters(cmd);
            con.Close();

            foreach (System.Data.SqlClient.SqlParameter param in cmd.Parameters)
            {
                string name = param.ParameterName.ToString().Substring(1);
                if (name == "RETURN_VALUE") continue;

                if (name == "StartRowIndex")
                {
                    param.Value = start;
                    continue;
                }

                if (name == "MaximumRows")
                {
                    param.Value = limit;
                    continue;
                }

                if (name == "SortField")
                {
                    param.Value = sort;
                    continue;
                }

                if (name == "SortDirection")
                {
                    param.Value = dir;
                    continue;
                }

                string extJsType = ExtJs.TypeConverter.ToExtJsType(param.SqlDbType);
                if (extJsType == "number")
                {
                    if (name.EndsWith("To")) param.Value = fp.getNumericTo(Regex.Replace(name, "To$", ""));
                    else if (name.EndsWith("From")) param.Value = fp.getNumericFrom(Regex.Replace(name, "From$", ""));
                    else if (name.EndsWith("Equal")) param.Value = fp.getNumericEq(Regex.Replace(name, "Equal$", ""));
                }
                else if (extJsType == "bool") param.Value = fp.getBool(name);
                else if (extJsType == "string")
                {
                    string name2find = name;
                    if (name.EndsWith("s")) name2find = name2find.Substring(0, name2find.Length - 1);
                    param.Value = fp.getList(name2find);
                }
                else if (extJsType == "date")
                {
                    if (name.EndsWith("To")) param.Value = fp.getDateTo(Regex.Replace(name, "To$", ""));
                    else if (name.EndsWith("From")) param.Value = fp.getDateFrom(Regex.Replace(name, "From$", ""));
                    else if (name.EndsWith("Equal")) param.Value = fp.getDateEq(Regex.Replace(name, "Equal$", ""));
                }
            }
        }

        public string GetJson()
        {
            this.da.Fill(this.ds);

            var response = new
            {
                total = Convert.ToInt32(ds.Tables[1].Rows[0][0]),
                success = true,
                message = "ok",
                metaData = new ExtJs.MetaData(ds.Tables[0], this.id_column_name, this.sort, this.dir),
                data = ds.Tables[0]
            };

            return Newtonsoft.Json.JsonConvert.SerializeObject(response, Newtonsoft.Json.Formatting.Indented, new Newtonsoft.Json.Converters.JavaScriptDateTimeConverter());
        }

        public override string ToString()
        {
            if (this.context.Request.QueryString["excel"] == null)
            {
                return this.GetJson();
            }
            else
            {
                this.cmd.Parameters["@StartRowIndex"].Value = 0;
                this.cmd.Parameters["@MaximumRows"].Value = 100000;

                this.da.Fill(this.ds);

                this.context.Response.Clear();
                this.context.Response.ContentType = "application/ms-excel";
                this.context.Response.AddHeader("Content-Disposition", "attachment;filename=report.xls");

                ExcelHelper helper = new ExcelHelper();
                string file = helper.ProcessToExcel(this.ds.Tables[0]);
                this.context.Response.WriteFile(file);

                this.context.Response.Flush();

                try
                {
                    File.Delete(file);
                }
                catch { }

                return file;
            }
        }
    }
}
```

Also there is TypeConverter helper:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Data;

namespace ExtJs
{
    public static class TypeConverter
    {
        private struct TypeMapEntry
        {
            public Type Type;
            public DbType DbType;
            public SqlDbType SqlDbType;
            public string ExtJsType;
            public TypeMapEntry(Type type, DbType dbType, SqlDbType sqlDbType, string extJsType)
            {
                this.Type = type;
                this.DbType = dbType;
                this.SqlDbType = sqlDbType;
                this.ExtJsType = extJsType;
            }
        };

        private static List<TypeMapEntry> _TypeList = new List<TypeMapEntry>();

        static TypeConverter()
        {
            _TypeList.Add(new TypeMapEntry(typeof(bool), DbType.Boolean, SqlDbType.Bit, "bool"));
            _TypeList.Add(new TypeMapEntry(typeof(byte), DbType.Double, SqlDbType.TinyInt, "number"));
            _TypeList.Add(new TypeMapEntry(typeof(DateTime), DbType.DateTime, SqlDbType.DateTime, "date"));
            _TypeList.Add(new TypeMapEntry(typeof(Decimal), DbType.Decimal, SqlDbType.Decimal, "number"));
            _TypeList.Add(new TypeMapEntry(typeof(double), DbType.Double, SqlDbType.Float, "number"));
            _TypeList.Add(new TypeMapEntry(typeof(Guid), DbType.Guid, SqlDbType.UniqueIdentifier, "string"));
            _TypeList.Add(new TypeMapEntry(typeof(Int16), DbType.Int16, SqlDbType.SmallInt, "number"));
            _TypeList.Add(new TypeMapEntry(typeof(Int32), DbType.Int32, SqlDbType.Int, "number"));
            _TypeList.Add(new TypeMapEntry(typeof(Int64), DbType.Int64, SqlDbType.BigInt, "number"));
            _TypeList.Add(new TypeMapEntry(typeof(string), DbType.String, SqlDbType.VarChar, "string"));
        }

        public static Type ToNetType(DbType dbType)
        {
            TypeMapEntry entry = Find(dbType);
            return entry.Type;
        }

        public static Type ToNetType(SqlDbType sqlDbType)
        {
            TypeMapEntry entry = Find(sqlDbType);
            return entry.Type;
        }

        public static Type ToNetType(string extJsType)
        {
            TypeMapEntry entry = Find(extJsType);
            return entry.Type;
        }

        public static DbType ToDbType(Type type)
        {
            TypeMapEntry entry = Find(type);
            return entry.DbType;
        }

        public static DbType ToDbType(SqlDbType sqlDbType)
        {
            TypeMapEntry entry = Find(sqlDbType);
            return entry.DbType;
        }

        public static DbType ToDbType(string extJsType)
        {
            TypeMapEntry entry = Find(extJsType);
            return entry.DbType;
        }

        public static SqlDbType ToSqlDbType(Type type)
        {
            TypeMapEntry entry = Find(type);
            return entry.SqlDbType;
        }

        public static SqlDbType ToSqlDbType(DbType dbType)
        {
            TypeMapEntry entry = Find(dbType);
            return entry.SqlDbType;
        }

        public static SqlDbType ToSqlDbType(string extJsType)
        {
            TypeMapEntry entry = Find(extJsType);
            return entry.SqlDbType;
        }

        public static string ToExtJsType(Type type)
        {
            TypeMapEntry entry = Find(type);
            return entry.ExtJsType;
        }

        public static string ToExtJsType(DbType dbType)
        {
            TypeMapEntry entry = Find(dbType);
            return entry.ExtJsType;
        }

        public static string ToExtJsType(SqlDbType sqlDbType)
        {
            TypeMapEntry entry = Find(sqlDbType);
            return entry.ExtJsType;
        }

        private static TypeMapEntry Find(Type type)
        {
            object retObj = null;
            for (int i = 0; i < _TypeList.Count; i++)
            {
                TypeMapEntry entry = _TypeList[i];
                if (entry.Type == type)
                {
                    retObj = entry;
                    break;
                }
            }
            if (retObj == null)
            {
                throw new ApplicationException("Referenced an unsupported Type");
            }
            return (TypeMapEntry)retObj;
        }

        private static TypeMapEntry Find(DbType dbType)
        {
            object retObj = null;
            for (int i = 0; i < _TypeList.Count; i++)
            {
                TypeMapEntry entry = (TypeMapEntry)_TypeList[i];
                if (entry.DbType == dbType)
                {
                    retObj = entry;
                    break;
                }
            }
            if (retObj == null)
            {
                throw new ApplicationException("Referenced an unsupported DbType");
            }
            return (TypeMapEntry)retObj;
        }

        private static TypeMapEntry Find(SqlDbType sqlDbType)
        {
            object retObj = null;
            for (int i = 0; i < _TypeList.Count; i++)
            {
                TypeMapEntry entry = (TypeMapEntry)_TypeList[i];
                if (entry.SqlDbType == sqlDbType)
                {
                    retObj = entry;
                    break;
                }
            }
            if (retObj == null)
            {
                throw new ApplicationException("Referenced an unsupported SqlDbType");
            }

            return (TypeMapEntry)retObj;
        }

        private static TypeMapEntry Find(string extJsType)
        {
            object retObj = null;
            for (int i = 0; i < _TypeList.Count; i++)
            {
                TypeMapEntry entry = (TypeMapEntry)_TypeList[i];
                if (entry.ExtJsType == extJsType)
                {
                    retObj = entry;
                    break;
                }
            }
            if (retObj == null)
            {
                throw new ApplicationException("Referenced an unsupported SqlDbType");
            }

            return (TypeMapEntry)retObj;
        }
    }
}
```
