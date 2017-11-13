---
layout: post
title: YUI DataTable Custom CellEditor

tags: [GUI, javascript, UI, YUI]
---

При создании интерактивных веб приложений очень часто приходиться иметь дело с формами состоящими из зарание не определенного числа элементов. Для того чтобы каждый раз не изобретать велосипед стоит обратить внимание на уже готовые решения, такие как библиотека YUI и ее класс DataTable.


Очень здорово если класс DataTable и его CellEditor’ы решат задачу сразу, но что делать если нужно создать custom celleditor для своих данных?Для начала необходимо подключить все необходимые для работы библиотеки (я в своем примере подключаю не минимизированные – для того чтобы можно было в случае необходимости проследить что где и как работает) и создать простую таблицу на основе данных.


```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<link rel="stylesheet" type="text/css" href="http://yui.yahooapis.com/2.8.0r4/build/fonts/fonts-min.css" />
<link rel="stylesheet" type="text/css" href="http://yui.yahooapis.com/2.8.0r4/build/datatable/assets/skins/sam/datatable.css" />
<script type="text/javascript" src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js"></script>
<script type="text/javascript" src="http://yui.yahooapis.com/2.8.0r4/build/element/element.js"></script>
<script type="text/javascript" src="http://yui.yahooapis.com/2.8.0r4/build/datasource/datasource.js"></script>
<script type="text/javascript" src="http://yui.yahooapis.com/2.8.0r4/build/datatable/datatable.js"></script>
</head>
<body class="yui-skin-sam">
<div id="languagesDiv"></div>
<script type="text/javascript">
    YAHOO.namespace('example');

    YAHOO.example.Data = {
        languages: [
            { label: 'english', value: 1 },
            { label: 'russian', value: 2 },
            { label: 'ukrainian', value: 3 }
        ],

        skills: [
            { label: 'little', value: 1 },
            { label: 'middle', value: 2 },
            { label: 'good', value: 3 }
        ],

        lng: [
            { language: 2, skill: 3 },
            { language: 3, skill: 3 }
        ]
    };

    YAHOO.example.getLabel = function(opts, val) {
        for (var i = 0; i < opts.length; i++) {
            if (opts[i].value == val) {
                return opts[i].label;
            }
        }
        return '- N/A -';
    }

    var formatLanguage = function (elCell, oRecord, oColumn, oData) {
        var langname = YAHOO.example.getLabel(YAHOO.example.Data.languages, oRecord.getData('language'));
        var langskill = YAHOO.example.getLabel(YAHOO.example.Data.skills, oRecord.getData('skill'));

        elCell.innerHTML = langname + ' (' + langskill + ')';
    }

    var myColumnDefs = [{formatter: formatLanguage, label: 'LANGUAGES', key: 'language'}];

    var myDataSource = new YAHOO.util.DataSource(YAHOO.example.Data.lng);
    myDataSource.responseType = YAHOO.util.DataSource.TYPE_JSARRAY;
    myDataSource.responseSchema = { fields: ['language', 'skill'] };

    var myDataTable = new YAHOO.widget.DataTable('languagesDiv', myColumnDefs, myDataSource, {});
</script>
</body>
</html>
```

Все делается точно так же как и в примерах документации к [YUI](http://developer.yahoo.com/yui/) виджету [DataTable](http://developer.yahoo.com/yui/docs/YAHOO.widget.DataTable.html).

<amp-img src="/images/wp/yui-datatable-custom-celleditor-step1.png" alt="step1" width="124" height="89"></amp-img>

Сами данные в моем случае хранятся в массиве `YAHOO.example.Data.lng`, массивы `YAHOO.example.Data.languages` и `YAHOO.example.Data.skills` – вспомогательные и служат для форматирования выводимых данных, а так же в будущем для генерации списков в редакторе.

Дальше самое интересное и сложное. Создание редактора началось с копирования класса [TextboxCellEditor](http://developer.yahoo.com/yui/docs/YAHOO.widget.TextboxCellEditor.html) и его постепенного переписывания.

Редактор наследует класс [BaseCellEditor](http://developer.yahoo.com/yui/docs/YAHOO.widget.BaseCellEditor.html) и обязан реализовать ряд методов, что очень хорошо видно в [исходниках](http://developer.yahoo.com/yui/docs/CellEditor.js.html).

```js
(function() {
    YAHOO.widget.LanguageCellEditor = function(oConfigs) {
        this._sId = "yui-languageceditor"
                + YAHOO.widget.BaseCellEditor._nCount++;
        YAHOO.widget.LanguageCellEditor.superclass.constructor.call(this,
                "language", oConfigs);
    };

    YAHOO.lang
            .extend(
                    YAHOO.widget.LanguageCellEditor,
                    YAHOO.widget.BaseCellEditor,
                    {
                        textbox : null,

                        renderForm : function() {
                            var elTextbox;
                            // Bug 1802582: SF3/Mac needs a form element
                            // wrapping the input
                            if (YAHOO.env.ua.webkit > 420) {
                                elTextbox = this
                                        .getContainerEl()
                                        .appendChild(
                                                document.createElement("form"))
                                        .appendChild(
                                                document.createElement("input"));
                            } else {
                                elTextbox = this.getContainerEl().appendChild(
                                        document.createElement("input"));
                            }
                            elTextbox.type = "text";
                            this.textbox = elTextbox;

                            // Save on enter by default
                            // Bug: 1802582 Set up a listener on each textbox to
                            // track on keypress
                            // since SF/OP can't preventDefault on keydown
                            YAHOO.util.Event.addListener(elTextbox, "keypress",
                                    function(v) {
                                        if ((v.keyCode === 13)) {
                                            // Prevent form submit
                                    YAHOO.util.Event.preventDefault(v);
                                    this.save();
                                }
                            }, this, true);

                            if (this.disableBtns) {
                                // By default this is no-op since enter saves by
                                // default
                                this.handleDisabledBtns();
                            }
                        },

                        move : function() {
                            this.textbox.style.width = this.getTdEl().offsetWidth
                                    + "px";
                            YAHOO.widget.LanguageCellEditor.superclass.move
                                    .call(this);
                        },

                        resetForm : function() {
                            this.textbox.value = YAHOO.lang.isValue(this.value) ? this.value
                                    .toString()
                                    : "";
                        },

                        focus : function() {
                            // Bug 2303181, Bug 2263600
                        this.getDataTable()._focusEl(this.textbox);
                        this.textbox.select();
                    },

                    getInputValue : function() {
                        return this.textbox.value;
                    }

                    });

    YAHOO.lang.augmentObject(YAHOO.widget.LanguageCellEditor,
            YAHOO.widget.BaseCellEditor);
})();
```

Все что я сделал:

  * первым делом это скопировал содержимое их исходников
  * удалил все что не касалось объявления TextboxCellEditor
  * переименовал `TextboxCellEditor` на `LanguageCellEditor`

Основной задачей всех этих манипуляций было получение своего редактора для дальнейшего его изменения.

Получилось вот так:

<amp-img src="/images/wp/yui-datatable-custom-celleditor-step2.png" alt="step2" width="140" height="121"></amp-img>

Уже сейчас все это дело работает, изменяя цифру мы тем самым меняем язык.

Собственно забавно получается, кода стало еще больше, а эффект все еще смешной, но есть обнадеживающий факт – у меня уже есть свой редактор который я могу изменять как угодно. Изучив немного его исходники стало понятно как он работает, но остался один не закрытый вопрос, как редактировать запись целиком (язык и уровень владения), а не её отдельную часть (язык). После изучения исходников класса `BaseCellEditor` нашел два места которые нужно изменить, собственно методы `attach` и `save`, в них идет основная работа с редактируемыми значениями и их нужно переопределить. Так же нужно в редакторе показывать два списка вместо текстового поля, большую часть кода подсмотрел из класса `DropdownCellEditor`.

```js
(function() {
    YAHOO.widget.LanguageCellEditor = function(oConfigs) {
        this._sId = "yui-languageceditor"
                + YAHOO.widget.BaseCellEditor._nCount++;
        YAHOO.widget.LanguageCellEditor.superclass.constructor.call(this,
                "language", oConfigs);
    };

    YAHOO.lang
            .extend(
                    YAHOO.widget.LanguageCellEditor,
                    YAHOO.widget.BaseCellEditor,
                    {
                        // CELL EDITOR OVERRIDEN METHOD, TO EDIT ENTIRE ROW
                        // RATHER THAT CELL
                        // START: DO NOT CHANGE THIS
                        attach : function(oDataTable, elCell) {
                            if (YAHOO.widget.LanguageCellEditor.superclass.attach
                                    .call(this, oDataTable, elCell) == false)
                                return false;
                            // CHANGE: rewrite current value with entire record
                            this.value = this._oRecord;
                            return true;
                        },

                        save : function() {
                            // Get new value
                            var inputValue = this.getInputValue();
                            var validValue = inputValue;

                            // Validate new value
                            if (this.validator) {
                                validValue = this.validator.call(this
                                        .getDataTable(), inputValue,
                                        this.value, this);
                                if (validValue === undefined) {
                                    if (this.resetInvalidData) {
                                        this.resetForm();
                                    }
                                    this.fireEvent("invalidDataEvent", {
                                        editor : this,
                                        oldData : this.value,
                                        newData : inputValue
                                    });
                                    YAHOO.log(
                                            "Could not save Cell Editor input due to invalid data "
                                                    + YAHOO.lang
                                                            .dump(inputValue),
                                            "warn", this.toString());
                                    return;
                                }
                            }

                            var oSelf = this;
                            var finishSave = function(bSuccess, oNewValue) {
                                var oOrigValue = oSelf.value;
                                if (bSuccess) {
                                    // Update new value
                                    oSelf.value = oNewValue;

                                    // CHANGED: we update entire row rather that
                                    // single cell
                                    // oSelf.getDataTable().updateCell(oSelf.getRecord(),
                                    // oSelf.getColumn(), oNewValue);
                                    oSelf.getDataTable().updateRow(
                                            oSelf.getRecord(), oNewValue);

                                    // Hide CellEditor
                                    oSelf.getContainerEl().style.display = "none";
                                    oSelf.isActive = false;
                                    oSelf.getDataTable()._oCellEditor = null;

                                    oSelf.fireEvent("saveEvent", {
                                        editor : oSelf,
                                        oldData : oOrigValue,
                                        newData : oSelf.value
                                    });
                                    YAHOO.log("Cell Editor input saved",
                                            "info", this.toString());
                                } else {
                                    oSelf.resetForm();
                                    oSelf.fireEvent("revertEvent", {
                                        editor : oSelf,
                                        oldData : oOrigValue,
                                        newData : oNewValue
                                    });
                                    YAHOO.log(
                                            "Could not save Cell Editor input "
                                                    + YAHOO.lang
                                                            .dump(oNewValue),
                                            "warn", oSelf.toString());
                                }
                                oSelf.unblock();
                            };

                            this.block();
                            if (YAHOO.lang.isFunction(this.asyncSubmitter)) {
                                this.asyncSubmitter.call(this, finishSave,
                                        validValue);
                            } else {
                                finishSave(true, validValue);
                            }
                        },
                        // END: DO NOT CHANGE THIS

                        // CELL EDITOR PROPERTIES
                        ddlLanguageName : null,
                        ddlLanguageSkill : null,
                        languageOptions : null,
                        languageSkillOptions : null,

                        // CELL EDITOR IMPLEMENTED METHODS
                        renderForm : function() {
                            var elLanguageName = this.getContainerEl()
                                    .appendChild(
                                            document.createElement("select"));
                            elLanguageName.style.zoom = 1;
                            this.ddlLanguageName = elLanguageName;

                            var elNAOption = document.createElement("option");
                            elNAOption.value = 0;
                            elNAOption.innerHTML = '- N/A -';
                            elLanguageName.appendChild(elNAOption);

                            if (YAHOO.lang.isArray(this.languageOptions)) {
                                var dropdownOption, elOption;
                                for ( var i = 0, j = this.languageOptions.length; i < j; i++) {
                                    dropdownOption = this.languageOptions[i];
                                    elOption = document.createElement("option");
                                    elOption.value = (YAHOO.lang
                                            .isValue(dropdownOption.value)) ? dropdownOption.value
                                            : dropdownOption;
                                    elOption.innerHTML = (YAHOO.lang
                                            .isValue(dropdownOption.label)) ? dropdownOption.label
                                            : dropdownOption;
                                    elOption = elLanguageName
                                            .appendChild(elOption);
                                }
                            }

                            var elLanguageSkill = this.getContainerEl()
                                    .appendChild(
                                            document.createElement("select"));
                            elLanguageSkill.style.zoom = 1;
                            this.ddlLanguageSkill = elLanguageSkill;

                            var elNAOption = document.createElement("option");
                            elNAOption.value = 0;
                            elNAOption.innerHTML = '- N/A -';
                            elLanguageSkill.appendChild(elNAOption);

                            if (YAHOO.lang.isArray(this.languageSkillOptions)) {
                                var dropdownOption, elOption;
                                for ( var i = 0, j = this.languageSkillOptions.length; i < j; i++) {
                                    dropdownOption = this.languageSkillOptions[i];
                                    elOption = document.createElement("option");
                                    elOption.value = (YAHOO.lang
                                            .isValue(dropdownOption.value)) ? dropdownOption.value
                                            : dropdownOption;
                                    elOption.innerHTML = (YAHOO.lang
                                            .isValue(dropdownOption.label)) ? dropdownOption.label
                                            : dropdownOption;
                                    elOption = elLanguageSkill
                                            .appendChild(elOption);
                                }
                            }

                            if (this.disableBtns) {
                                this.handleDisabledBtns();
                            }
                        },

                        resetForm : function() {
                            this.ddlLanguageName.value = YAHOO.lang
                                    .isValue(this.value.getData('language')) ? this.value
                                    .getData('language')
                                    : 0;
                            this.ddlLanguageSkill.value = YAHOO.lang
                                    .isValue(this.value.getData('skill')) ? this.value
                                    .getData('skill')
                                    : 0;
                        },

                        getInputValue : function() {
                            var res = {};
                            res.language = this.ddlLanguageName.value;
                            res.skill = this.ddlLanguageSkill.value;
                            return res;
                        }
                    });

    YAHOO.lang.augmentObject(YAHOO.widget.LanguageCellEditor,
            YAHOO.widget.BaseCellEditor);
})();
```

<amp-img src="/images/wp/yui-datatable-custom-celleditor-step3.png" alt="step3" width="168" height="123"></amp-img>

Основная часть задачи уже решена, в методах `attach` и `save` комментариями отмечены места которые были изменены, для достижения результата, теперь наш редактор гоняет запись целиком, вместо того чтобы гонять значение одной ячейки.

Следующим логичным этапом будет добавление возможности добавления и удаления записей, что делается весьма просто.

<amp-img src="/images/wp/yui-datatable-custom-celleditor-step4.png" alt="step4" width="164" height="94"></amp-img>

Я просто добавил соответствующие ссылки в шапку таблицы и к каждой записи:

```js
var formatLanguage = function(elCell, oRecord, oColumn, oData){
    var langname = YAHOO.example.getLabel(YAHOO.example.Data.languages, oRecord.getData('language'));
    var langskill = YAHOO.example.getLabel(YAHOO.example.Data.skills, oRecord.getData('skill'));

    elCell.innerHTML = '<a href="javascript:void(0)" onclick="deleteLanguage(event, \'' + oRecord.getId() + '\')" style="text-decoration:none;color:#f00;">x</a> ' + langname + ' (' + langskill + ')';
}

var myColumnDefs = [{
    label: 'LANGUAGES <button onclick="addLanguage(event)">add</button>',
    key: 'language',
    formatter: formatLanguage,
    editor: new YAHOO.widget.LanguageCellEditor({
        languageOptions: YAHOO.example.Data.languages,
        languageSkillOptions: YAHOO.example.Data.skills
    }),
}];
```

и добавил две новые функции:

```js
function addLanguage(e) {
    // PREVENT BLUR EVENTS
    e = e || window.event;
    YAHOO.util.Event.stopEvent(e);

    var r = {};
    r.language = 0;
    r.skill = 0;

    myDataTable.addRow(r);
    myDataTable.showCellEditor(myDataTable.getLastTrEl().cells[0]);
}

function deleteLanguage(e, recordId) {
    // PREVENT SHOW CELL EDITOR EVENT
    e = e || window.event;
    YAHOO.util.Event.stopEvent(e);

    myDataTable.deleteRow(recordId);
}
```

Теперь уже совсем то что нужно, но не для меня, мне нужно добиться совсем другого чем я имею сейчас. Первое что мне нужно это чтобы окно редактирования не плавало по странице, а как бы заменяло ячейку, естественно так глубоко я не залез, и пошел простым путем, все что я сделал, это метод

```js
fixTdHeight: function(bSetHeightToAuto){
    this.getTdEl().style.height = (bSetHeightToAuto) ? 'auto' : this.getContainerEl().offsetHeight + 'px';
}
```

который делал высоту ячейки таблицы такой же как и высота редактора, и вызывал этот метод когда окно редактора появлялось либо пряталось (методы: `save`, `move`, `cancel`, `show`).

```js
move: function(){
    this.getContainerEl().style.width = (this.getTdEl().offsetWidth - 14) + "px";
    YAHOO.widget.LanguageCellEditor.superclass.move.call(this);
},

show: function(){
    YAHOO.widget.LanguageCellEditor.superclass.show.call(this);
    this.fixTdHeight(false);
},

cancel: function(){
    YAHOO.widget.LanguageCellEditor.superclass.cancel.call(this);
    this.fixTdHeight(true);
},
```

<amp-img src="/images/wp/yui-datatable-custom-celleditor-step5.png" alt="step5" width="165" height="181"></amp-img>

Вот теперь это уже почти то что мне нужно, но остался еще один важный момент – валидация. Для его реализации пришлось перелопатить целую кучу кода, собственно тут уже каждый сам для себя должен определить что и как ему нужно.  Дело в том что поведение редактора по умолчанию меня совсем не устраивает, так например, нажав на кнопку добавления записи, мы добавляем новую запись со значениями равными нулю, мне бы хотелось чтобы при отмене эта запись удалялась. Так же при попытке сохранить значение не удовлетворяющее правилам валидации, редактор просто закрывается и не сохраняет значение – мне нужно оставлять его открытым и показывать уведомление об ошибке.

По концовке получился вот такой контрол:

<amp-img src="/images/wp/yui-datatable-custom-celleditor-result.png" alt="result" width="162" height="205"></amp-img>

Его исходники: [yui-custom-celleditor](/images/wp/yui-custom-celleditor.html)

Вот что из этого всего получилось в реальных условиях:

<a href="http://www.youtube.com/watch?v=hMQz_MLIxQw"><amp-img src="http://img.youtube.com/vi/hMQz_MLIxQw/0.jpg" alt="video" width="480" height="360"></amp-img></a>

К сожалению посмотреть пощупать так просто не получится, придеться довольствоваться видео
