---
layout: post
title: Создание плагина EditArea для ModX

tags: [modx, php, editarea]
---

Аннотация
---------

В ModX очень не удобное редактирование чанков и сниппетов. Админка этого движка предлагает заниматься этим в обычном текстовом поле еще и с совсем мелким шрифтом – что очень сильно напрягает.

Задача
------

Необходимо сделать так чтобы выполнять все эти работы можно было без бинокля

Решение
-------

Для решения этой задачи будем использовать бесплатный редактор [EditArea](http://www.cdolivet.com/index.php?page=editArea), который покрывает все пожелания которые только можно придумать.

О том как прикрутить куда либо EditArea, я уже успел написать. Плагин уже тоже готов, и лежит [здесь](http://code.google.com/p/modxeditarea/).

Итак как и любой плагин, этот должен вешаться на события, естественно события связанные с рендером форм для чанков и сниппетов, а заодно и плагинов `OnChunkFormPreRender`, `OnSnipFormRender`, `OnPluginFormRender` (почему для чанков выбрано событие PreRender раскажу чуть позже).

Естественно что писать с нуля никто ничего не собирался и первое что было сделано, это изучение встроенного плагина tinyMCE, по результатам которого выяснилось что можно еще и зарегистрировать сам редактор в системе, для этого надо повеситься на события `OnRichTextEditorRegister`, `OnRichTextEditorInit`.

Не долго думая, код был скопирован, получилось вот такое:

```php
$e = &$modx->Event;
switch ($e->name) {
    case "OnRichTextEditorRegister":
        $e->output("EditArea");
        break;

    case "OnRichTextEditorInit":
        if($editor=="EditArea") {
            $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/edit_area/edit_area_full.js"></script>');
            $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/ea_functions.js"></script>');
            foreach($elements as $element) {
                $e->output('<script type="text/javascript">initEditArea("'.$element.'", "'.$modx->config['manager_language'].'", "html")</script>');
            }
        }
        break;

    default :
        return;
        break;
}
```

Что мы делаем:

`OnRichTextEditorRegister` – это событие вызывается когда система запрашивает список установленных редакторов, собственно тут наша задача, просто обозначить что мы существуем.

`OnRichTextEditorInit` – а вот тут уже по интереснее, это событие наступает когда юзер выбирает редактор, мы робко проверяем что выбрали именно нас и лепим необходимые яваскрипты в вывод. Так же нам передают массив элементов на которые нужно повесить редактор, мы для каждого из них вызываем функцию инициализации редактора, в которую передаем имя элемента, язык админки (это я уже потом добавил, для красоты пущей) и синтаксис подсветки по умолчанию (это нам надо потому что по умолчанию у чанков будет html, а вот у сниппетов и плагинов – php).

Собственно содержимое файла `ea-functions.js`:

```js
function onEditAreaChanged(id) {
    document.getElementById(id).value = editAreaLoader.getValue(id);
}

function getLangName( lang ) {
    var res = '';

    switch(lang)
    {
        case 'english':     res = 'en'; break;
        case 'russian':     res = 'ru'; break;
        case 'russian-UTF8':    res = 'ru'; break;
        default: res = 'en';
    }

    return res;
}

function initEditArea(el_name, lang, syntax) {
    var elements = document.getElementsByName(el_name);
    if(!elements) return;
    var el = elements.item(0);

    if(!el.id) el.id = el.name;

    editAreaLoader.init({
        id: el.id,
        change_callback: 'onEditAreaChanged',
        language: 'ru',
        start_highlight: true,
        allow_resize: 'y',
        allow_toggle: true,
        word_wrap: true,
        syntax: syntax,
        toolbar: 'undo, redo, |, select_font, |, syntax_selection, |, change_smooth_selection, highlight, reset_highlight, word_wrap',
        syntax_selection_allow: 'css,html,js,php,xml,sql'
    });
}
```

Собственно основная функция `initEditArea` только тем и занимается что создает редактор (там в самом начале еще небольшой шахер махер делается, потому что по умолчанию у текстовых полей нет id’ек, а редактору они нужны).  Описывать остальные функции по моему вообще нет никакого смысле так как они ну до безобразия простые.

И все заработало, теперь на страницах где есть выбор редактора – можно выбрать EditArea и будет счастье, я уже думал радоваться но не тут то было, оказалось что на страницах снипетов и плагинов, редактор не предпологается, поетому пришлось воевать дальше.

Но, для начала, нужно было решить еще одну вещь, на страницах чанков по умолчанию редактор не используется, а я хочу чтобы использовался, вот именно для этого мы и вешаемся на событие `OnChunkFormPrerender`.

```php
case "OnChunkFormPrerender":
    global $which_editor;
    $which_editor = 'EditArea';
    break;
```

Вот так все просто оказалось. Как нашел – в файле `manager/actions/mutate_htmlsnippet.dynamic.php` – там видно что при генерации выпадайки с выбором редактора используется переменная `$which_editor`.

Для сниппетов и плагинов, код одинаковый и выглядит так:

```php
case "OnSnipFormRender":
case "OnPluginFormRender":
    $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/edit_area/edit_area_full.js"></script>');
    $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/ea_functions.js"></script>');
    $e->output('<script type="text/javascript">initEditArea("post", "'.$modx->config['manager_language'].'", "php")</script>');
    break;
```

Мы просто добавляем наши скрипты, и вызываем инициализацию редактора.

Вот собственно и все.

Исходник плагина
----------------

```php
/*    EVENTS
    OnRichTextEditorRegister
    OnRichTextEditorInit
    OnChunkFormPrerender
    OnSnipFormRender
    OnPluginFormRender
*/
$e = &$modx->Event;
switch ($e->name) {
    case "OnRichTextEditorRegister":
        $e->output("EditArea");
        break;

    case "OnRichTextEditorInit":
        if($editor=="EditArea") {
            $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/edit_area/edit_area_full.js"></script>');
            $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/ea_functions.js"></script>');
            foreach($elements as $element) {
                $e->output('<script type="text/javascript">initEditArea("'.$element.'", "'.$modx->config['manager_language'].'", "html")</script>');
            }
        }
        break;

    case "OnChunkFormPrerender":
        global $which_editor;
        $which_editor = 'EditArea';
        break;

    case "OnSnipFormRender":
    case "OnPluginFormRender":
        $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/edit_area/edit_area_full.js"></script>');
        $e->output('<script type="text/javascript" src="'.$modx->config['base_url'].'assets/plugins/editarea/ea_functions.js"></script>');
        $e->output('<script type="text/javascript">initEditArea("post", "'.$modx->config['manager_language'].'", "php")</script>');
        break;

    default :
        return; // stop here - this is very important.
        break;
}
```

JavaScript
----------

```js
function onEditAreaChanged(id) {
    document.getElementById(id).value = editAreaLoader.getValue(id);
}

function getLangName( lang ) {
    var res = '';

    switch(lang)
    {
        case 'english':     res = 'en'; break;
        case 'russian':     res = 'ru'; break;
        case 'russian-UTF8':    res = 'ru'; break;
        default: res = 'en';
    }

    return res;
}

function initEditArea(el_name, lang, syntax) {
    var elements = document.getElementsByName(el_name);
    if(!elements) return;
    var el = elements.item(0);

    if(!el.id) el.id = el.name;

    editAreaLoader.init({
        id: el.id,
        change_callback: 'onEditAreaChanged',
        language: 'ru',
        start_highlight: true,
        allow_resize: 'y',
        allow_toggle: true,
        word_wrap: true,
        syntax: syntax,
        toolbar: 'undo, redo, |, select_font, |, syntax_selection, |, change_smooth_selection, highlight, reset_highlight, word_wrap',
        syntax_selection_allow: 'css,html,js,php,xml,sql'
    });
}
```
