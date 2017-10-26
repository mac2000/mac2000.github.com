---
layout: post
title: YUI RecordSet Events

tags: [javascript, YUI, yahoo, recordset, createevent, subscribe, fireevent]
---

В библиотеке [YUI](http://developer.yahoo.com/yui/) есть отличный класс [RecordSet](http://developer.yahoo.com/yui/docs/YAHOO.widget.RecordSet.html) который является эдаким декоратором вокруг массива данных.

Этот класс может очень сильно пригодиться при создании интерактивных веб приложений, у которых имеются формы с неопределенным количеством элементов.Пример использования:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<script type="text/javascript" src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js"></script>
<script type="text/javascript" src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js"></script>
<script type="text/javascript" src="http://yui.yahooapis.com/2.8.0r4/build/datatable/datatable-min.js"></script>
<script type="text/javascript">
    YAHOO.namespace('example');

    YAHOO.example.Data = [
            { id: 1, value: 'keyword1' },
            { id: 2, value: 'keyword2' },
            { id: 3, value: 'keyword3' },
            { id: 4, value: 'keyword4' }
        ];

    YAHOO.example.myRecordSet =  new YAHOO.widget.RecordSet(YAHOO.example.Data);

    YAHOO.example.myRecordSet.createEvent('recordDeleteEvent');
    YAHOO.example.myRecordSet.subscribe('recordDeleteEvent', function (oArgs) {
        alert('recordDeleteEvent\r\n' + YAHOO.lang.dump(oArgs));
    });

    YAHOO.example.myRecordSet.createEvent('recordAddEvent');
    YAHOO.example.myRecordSet.subscribe('recordAddEvent', function (oArgs) {
        alert('recordAddEvent\r\n' + YAHOO.lang.dump(oArgs));
    });

    YAHOO.example.myRecordSet.createEvent('recordValueUpdateEvent');
    YAHOO.example.myRecordSet.subscribe('recordValueUpdateEvent', function (oArgs) {
        alert('recordValueUpdateEvent\r\n' + YAHOO.lang.dump(oArgs));
    });

    function showRecordsInRecordSet() {
        alert(YAHOO.lang.dump(YAHOO.example.myRecordSet.getRecords()));
    }
    function deleteRecordByIndex(index) {
        YAHOO.example.myRecordSet.deleteRecord(index);
    }
    function addRecord() {
        var r = new YAHOO.widget.Record();
        r.setData('id', 5);
        r.setData('value', 'hello');
        YAHOO.example.myRecordSet.addRecord(r);
    }
    function changeRecord() {
        YAHOO.example.myRecordSet.updateRecordValue(0, 'value', 'hello world');
    }
</script>
</head>
<body>
<a href="javascript:void(0)" onclick="showRecordsInRecordSet()">show records</a><br />
<a href="javascript:void(0)" onclick="deleteRecordByIndex(0)">delete firs record</a><br />
<a href="javascript:void(0)" onclick="addRecord()">add record</a><br />
<a href="javascript:void(0)" onclick="changeRecord()">change record</a>
</body>
</html>
```

Теперь на странице есть **RecordSet** который генерирует события при изменении данных.

Примечание: Сам по себе **RecordSet** не генерирет собития, поэтому для их использования необходимо вызывать метод регистрации событий **createEvent**.
