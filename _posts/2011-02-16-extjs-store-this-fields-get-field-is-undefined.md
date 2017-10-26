---
layout: post
title: ExtJs store this.fields.get(field) is undefined

tags: [applysort, ext, extjs, javascript, js, jsonreader, jsonstore, metadata, sortinfo, store]
---

Here is simple example of store:

```js
var cityStore = new Ext.data.JsonStore({
    url: '<%= ResolveUrl("~/json/Dictionaries/Cities.ashx")%>',
    //remoteSort: true,
    autoLoad: true
});
```

And here is responce from cities.ashx:

```json
{
  "total": 34,
  "metaData": {
    "idProperty": "Id",
    "root": "data",
    "totalProperty": "total",
    "successProperty": "success",
    "messageProperty": "message",
    "sortInfo": "Id",
    "direction": "ASC",
    "fields": [
      {
        "name": "Id",
        "type": "number"
      },
      {
        "name": "Name",
        "type": "string"
      },
      {
        "name": "Count",
        "type": "number"
      }
    ]
  },
  "data": [
    {
      "Id": 5,
      "Name": "Винница",
      "Count": 0
    },
    {
      "Id": 30,
      "Name": "Горловка",
      "Count": 0
    },
    ...
  ],
  "success": true,
  "message": "ok"
}
```

If u will not add remoteSort:true u will get error:

```
this.fields.get(field) is undefined
```
