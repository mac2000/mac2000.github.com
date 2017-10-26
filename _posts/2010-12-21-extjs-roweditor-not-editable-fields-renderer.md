---
layout: post
title: ExtJS RowEditor not editable fields renderer

tags: [extjs, javascript]
---

RowEditor  не использует рендерер таблицы, а вызывает `Ext.form.DisplayField`, для  того чтобы исправить ситуацию, необходимо изменить метод `startEditing` в файле `RowEditor.js`

```js
startEditing: function(rowIndex, doFocus) {
    --//--
            for(var i = 0, len = cm.getColumnCount(); i < len; i++) {
val = this.preEditValue(record, cm.getDataIndex(i));
f = fields[i];

/////////////////////////////////////////////////////

var column = cm.getColumnById(cm.getColumnId(i));
                ed = column.getEditor();
if(!ed){
                val = column.renderer.call(column, val, {}, record);
            }

/////////////////////////////////////////////////////

f.setValue(val);

this.values[f.id] = Ext.isEmpty(val) ? '' : val;
            }
            --//--
}
```

так же для того чтобы не вылазило боков при редактировании необходимо подправить метод `stopEditing`

```js
stopEditing : function(saveChanges){
    --//--
        for(var i = 0, len = cm.getColumnCount(); i < len; i++){

/////////////////////////////////////////////////////

var column = cm.getColumnById(cm.getColumnId(i));
ed = column.getEditor();
if(!cm.isHidden(i) && ed){

/////////////////////////////////////////////////////

//if(!cm.isHidden(i)){
var dindex = cm.getDataIndex(i);
if(!Ext.isEmpty(dindex)){
            --//--
}
```
