---
layout: post
title: Extend YUI

tags: [javascript, oop, YUI]
---

Создание собственного класса на базе базового элемента библиотеки [YUI](http://developer.yahoo.com/yui/2/). Использование атрибутов класса, событий их изменения, а также собственных событий.

Extend YAHOO.util.Element
-------------------------

Для начала необходимо определить пространство имен, для того чтобы потом не вылезло боков с другими классами.

```js
YAHOO.namespace('example');
```

Дальше собственно определяем класс:

```js
YAHOO.example.MyClass = function (el, oConfigs) {
    YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
};
YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);
```

Здесь мы объявляем класс, конструктор которого принимает элемент в котором будет отображаться наш объект, это может быть строка с ID элемента на странице, либо сам элемент, и объект содержащий пары, ключ-значение параметров (о параметрах чуть дальше).

Класс наследует [YAHOO.util.Element](http://developer.yahoo.com/yui/docs/YAHOO.util.Element.html) и первым делом вызывает его конструктор.

Основа, готова, для того чтобы начать её ковырять.

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js" ></script>
        <script src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js" ></script>
        <script type="text/javascript">
            YAHOO.namespace('example');

            YAHOO.example.MyClass = function (el, oConfigs) {
                YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);

            var my = null;
            YAHOO.util.Event.onDOMReady(function(){
                my = new YAHOO.example.MyClass('mydiv');
            });
        </script>
    </head>
    <body>
        <div id="mydiv"></div>
    </body>
</html>
```

Аттрибуты класса
----------------

Для того чтобы определить свои атрибуты необходимо переопределить метод [initA ttributes](http://developer.yahoo.com/yui/docs/YAHOO.util.Element.html#method_initAttributes).

Выглядеть, в самом простом случае, это может вот так:

```js
YAHOO.example.MyClass.prototype.initAttributes = function (oConfigs) {
    YAHOO.example.MyClass.superclass.initAttributes.call(this, oConfigs);
    this.setAttributeConfig('prop1');
}
```

Для доступа к параметру используются методы `get` и `set` объекта созданного объекта. Так же параметр можно задать при создании объекта.

```js
my = new YAHOO.example.MyClass('mydiv', {prop1 : 5});
alert(my.get('prop1'));
my.set('prop1', 10);
alert(my.get('prop1'));
```

Далее полный исходник того что получилось, чтобы не потерять ход мысли:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js" ></script>
        <script src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js" ></script>
        <script type="text/javascript">
            YAHOO.namespace('example');

            YAHOO.example.MyClass = function (el, oConfigs) {
                YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);

            YAHOO.example.MyClass.prototype.initAttributes = function (oConfigs) {
                YAHOO.example.MyClass.superclass.initAttributes.call(this, oConfigs);

                this.setAttributeConfig('prop1');
            }

            var my = null;
            YAHOO.util.Event.onDOMReady(function(){
                my = new YAHOO.example.MyClass('mydiv', {prop1 : 5});
            });
        </script>
    </head>
    <body>
        <div id="mydiv"></div>
        <a href="javascript:void(0)" onclick="alert(my.get('prop1'))">getProp1</a> | <a href="javascript:void(0)" onclick="my.set('prop1', 5)">set prop1 to 5</a> | <a href="javascript:void(0)" onclick="my.set('prop1', 10)">set prop1 to 10</a>
    </body>
</html>
```

Параметры атрибутов
-------------------

Подробно все параметры расписаны на странице с официальной [докой](http://developer.yahoo.com/yui/docs/YAHOO.util.Attribute.html). Я рассматриваю только те что потенциально нужны мне.

  * `method` – функция которая будет вызвана при изменении атрибута, применяется для, к примеру, ререндеринга контрола при изменении атрибута
  * `owner` – я выставляю в this чтобы в функции объявленной в method через this было видно сам обьект, а не атрибут.
  * `readOnly` – из названия ясно что оно и для чего, удобно использовать для HTML элементов контролла. Если указать атрибут как readOnly – то задать его при создании объекта будет не возможно.
  * `validator` – функция получающая на вход значение, которое должна проверить и вернуть true либо false.
  * `value` – значение по умолчанию (перезаписывается тем что указано в параметрах, при создании объекта).
  * `writeOnce` – отличная вещь, позволяет разрешить записывать параметр всего один раз (в частности при создании объекта)

Пару примеров:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js" ></script>
        <script src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js" ></script>
        <script type="text/javascript">
            YAHOO.namespace('example');

            YAHOO.example.MyClass = function (el, oConfigs) {
                YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);

            YAHOO.example.MyClass.prototype.initAttributes = function (oConfigs) {
                YAHOO.example.MyClass.superclass.initAttributes.call(this, oConfigs);

                this.setAttributeConfig('prop1');
                this.setAttributeConfig('readOnlyProp', {readOnly : true, value : 5});
                this.setAttributeConfig('writeOnceProp', {writeOnce : true, value : 1});
                this.setAttributeConfig('validProp', {
                    validator: function(value) {
                        alert('validProp validator called for '+value);
                        return YAHOO.lang.isNumber(value);
                    }
                });
                this.setAttributeConfig('methodProp', {
                    method: this.methodProp
                });
            }

            YAHOO.example.MyClass.prototype.methodProp = function (value, key) {
                alert('methodProp called, after validating and before change of ' + key + ' to new value: '+value + '\r\nprop1 is: '+this.get('prop1'));
            }

            var my = null;
            YAHOO.util.Event.onDOMReady(function(){
                my = new YAHOO.example.MyClass('mydiv', {prop1 : 5, writeOnceProp: 5});
            });
        </script>
    </head>
    <body>
        <div id="mydiv"></div>
        <a href="javascript:void(0)" onclick="alert(my.get('prop1'))">getProp1</a> | <a href="javascript:void(0)" onclick="my.set('prop1', 5)">set prop1 to 5</a> | <a href="javascript:void(0)" onclick="my.set('prop1', 10)">set prop1 to 10</a>
        <br />
        <a href="javascript:void(0)" onclick="alert(my.get('readOnlyProp'))">getReadOnlyProp</a> | <a href="javascript:void(0)" onclick="my.set('readOnlyProp', 25)">set readOnlyProp to 25</a>
        <br />
        <a href="javascript:void(0)" onclick="alert(my.get('writeOnceProp'))">getWriteOnceProp</a> | <a href="javascript:void(0)" onclick="my.set('writeOnceProp', 25)">set writeOnceProp to 25</a>
        <br />
        <a href="javascript:void(0)" onclick="alert(my.get('validProp'))">getValidProp</a> | <a href="javascript:void(0)" onclick="my.set('validProp', 25)">set validProp to 25</a> | <a href="javascript:void(0)" onclick="my.set('validProp', 'zzz')">set validProp to 'zzz'</a>
        <br />
        <a href="javascript:void(0)" onclick="alert(my.get('methodProp'))">getMethodProp</a> | <a href="javascript:void(0)" onclick="my.set('methodProp', 25)">set methodProp to 25</a>
    </body>
</html>
```

Небольшое замечание, value для аттрибутов правильно устанавливать вот так:

```js
value: oConfigs.Keywords || new Array()

value: oConfigs.Formatter || function (elCell, oRecord, oColumn, oData) {
    elCell.innerHTML = '<img src="/Theme/img/cross_little_grey_icon.gif" alt="x" /> ' + oData;
}

value: oConfigs.ColumnDefs || [{
    key: 'Name',
    formatter: this.get('Formatter')
}]
```

fireBeforeChangeEvent и fireChangeEvent
---------------------------------------

Теперь, после того как определены атрибуты, можно подписаться на события их изменений, которые автоматом были созданны.

Вот пример использования событий:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js" ></script>
        <script src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js" ></script>
        <script type="text/javascript">
            YAHOO.namespace('example');

            YAHOO.example.MyClass = function (el, oConfigs) {
                YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);

            YAHOO.example.MyClass.prototype.initAttributes = function (oConfigs) {
                YAHOO.example.MyClass.superclass.initAttributes.call(this, oConfigs);

                this.setAttributeConfig('prop1');
            }

            var my = null;
            YAHOO.util.Event.onDOMReady(function(){
                my = new YAHOO.example.MyClass('mydiv', {prop1 : 5});

                my.subscribe('beforeProp1Change', function (oArgs) {
                    var type = oArgs.type;
                    var prevValue = oArgs.prevValue;
                    var newValue = oArgs.newValue;

                    alert('BEFORE Prop1 ' + prevValue + ' > ' + newValue + ' (*if newValue 5 - false)');
                    if (newValue == 5) return false; //value will not be set
                    return true;
                });

                my.subscribe('prop1Change', function (oArgs) {
                    var type = oArgs.type;
                    var prevValue = oArgs.prevValue;
                    var newValue = oArgs.newValue;

                    alert('AFTER Prop1 ' + prevValue + ' > ' + newValue);
                    return true;
                });
            });
        </script>
    </head>
    <body>
        <div id="mydiv"></div>
        <a href="javascript:void(0)" onclick="alert(my.get('prop1'))">getProp1</a> | <a href="javascript:void(0)" onclick="my.set('prop1', 1)">set prop1 to 1</a> | <a href="javascript:void(0)" onclick="my.set('prop1', 5)">set prop1 to 5</a> | <a href="javascript:void(0)" onclick="my.set('prop1', 10)">set prop1 to 10</a>
    </body>
</html>
```

Единственное на что здесь стоит обратить внимание – так это на правило формирования названия события, при событии изменения к актирбудут просто добавляется слово `Change`, при событии `BeforeAttributeChange` к атрибуту с обоих сторон добавляеются слова `Before` и `Change`, а первая буква атрибута переводиться в верхний регистр.

Custom Events
-------------

Теперь остался еще один небольшой пример который надо рассмотреть – custom events для класса. На самом деле все оказалось очень просто, все что нужно, это создать событие (метод [createEvent](http://developer.yahoo.com/yui/docs/YAHOO.util.EventProvider.html#method_createEvent)) и в необходимых местах вызвать его (метод [fireEvent](http://developer.yahoo.com/yui/docs/YAHOO.util.EventProvider.html#method_fireEvent)).

Далее небольшой пример показывающий как это реализовать:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js" ></script>
        <script src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js" ></script>
        <script type="text/javascript">
            YAHOO.namespace('example');

            YAHOO.example.MyClass = function (el, oConfigs) {
                YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);

            YAHOO.example.MyClass.prototype.initAttributes = function (oConfigs) {
                YAHOO.example.MyClass.superclass.initAttributes.call(this, oConfigs);

                this.createEvent('myCustomEvent');
            }

            YAHOO.example.MyClass.prototype.someMethod = function () {
                this.fireEvent('myCustomEvent', { instance: this, el: this.get('element'), hello: 'hello world' });
            }

            var my = null;
            YAHOO.util.Event.onDOMReady(function(){
                my = new YAHOO.example.MyClass('mydiv');

                my.subscribe('myCustomEvent', function (oArgs) {
                    var instance = oArgs.instance;
                    var el = oArgs.el;
                    var hello = oArgs.hello;
                    alert('myCustomEvent occured in control ' + el.id);
                });
            });
        </script>
    </head>
    <body>
        <div id="mydiv"></div>
        <a href="javascript:void(0)" onclick="my.someMethod()">call someMethod</a>
    </body>
</html>
```

Собственно говоря вот и вся наука рассширения классов в YUI.

Нет, еще не все.

Наследование в YUI
------------------

Тут все точно так же как и при создании класса, за исключением того что наследуемся не от `YAHOO.util.Element`, вот простенький пример:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js" ></script>
        <script src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js" ></script>
        <script type="text/javascript">
            YAHOO.namespace('example');
            // CLASS 1 /////////////////////////////////////////
            YAHOO.example.MyClass = function (el, oConfigs) {
                YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);

            YAHOO.example.MyClass.prototype.initAttributes = function (oConfigs) {
                YAHOO.example.MyClass.superclass.initAttributes.call(this, oConfigs);
                this.setAttributeConfig('prop1');
            }
            // CLASS 2 /////////////////////////////////////////
            YAHOO.example.MySecondClass = function (el, oConfigs) {
                YAHOO.example.MySecondClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MySecondClass, YAHOO.example.MyClass);
            ///////////////////////////////////////////
            var my = null;
            YAHOO.util.Event.onDOMReady(function(){
                my = new YAHOO.example.MySecondClass('mydiv', {prop1 : 5});
            });
        </script>
    </head>
    <body>
        <div id="mydiv"></div>
        <a href="javascript:void(0)" onclick="alert(my.get('prop1'))">getProp1</a>
    </body>
</html>
```

Композиция
----------

Склеиваем два класса

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <script src="http://yui.yahooapis.com/2.8.0r4/build/yahoo-dom-event/yahoo-dom-event.js" ></script>
        <script src="http://yui.yahooapis.com/2.8.0r4/build/element/element-min.js" ></script>
        <script type="text/javascript">
            YAHOO.namespace('example');
            // CLASS 1 /////////////////////////////////////////
            YAHOO.example.MyClass = function (el, oConfigs) {
                YAHOO.example.MyClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MyClass, YAHOO.util.Element);

            YAHOO.example.MyClass.prototype.someMethodFromMyClass = function () {
                alert('someMethodFromMyClass');
            }
            // CLASS 2 /////////////////////////////////////////
            YAHOO.example.MySecondClass = function (el, oConfigs) {
                YAHOO.example.MySecondClass.superclass.constructor.call(this, el, oConfigs);
            };
            YAHOO.extend(YAHOO.example.MySecondClass, YAHOO.util.Element);
            YAHOO.lang.augmentProto(YAHOO.example.MySecondClass, YAHOO.example.MyClass);

            YAHOO.example.MySecondClass.prototype.someMethodFromMySecondClass = function () {
                alert('someMethodFromMySecondClass');
                this.someMethodFromMyClass();
            }
            ///////////////////////////////////////////
            var my = null;
            YAHOO.util.Event.onDOMReady(function(){
                my = new YAHOO.example.MySecondClass('mydiv', {prop1 : 5});
            });
        </script>
    </head>
    <body>
        <div id="mydiv"></div>
        <a href="javascript:void(0)" onclick="my.someMethodFromMySecondClass()">call someMethodFromMySecondClass</a>
    </body>
</html>
```

подробней про это дело можно прочесть [здесь](http://developer.yahoo.com/yui/docs/YAHOO.lang.html).
