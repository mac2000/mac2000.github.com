---
layout: post
title: Editarea – wysiwyg для кода

tags: [editarea, javascript, wysiwyg]
---

В таких CMS, как ModX, редактирование внутренностей создаваемого сайта редактируется прямо в вебе из админки сайта, что иногда весьма не удобно, особенно с их темой и мелким шрифтом.

Случайно наткнулся на проект [EditArea](http://www.cdolivet.com/index.php?page=editArea), который в состоянии решить подобные проблемы.EditArea – эдакий TinyMCE, только для правки кода (HTML, CSS и т.п.).

![screenshot](/images/wp/editareascreenshot.png)

Как видно из скриншота ([больше примеров](http://www.cdolivet.com/editarea/editarea/exemples/exemple_full.html) на сайте разработчиков), эта штуковина нормально воспринимает Tab’ы, делает подсветку синтаксиса, может менять размер шрифтов и тип редактируемого файла.

Вобщем все чего душа может пожелать.

Использование так же очень простое, для этого необходимо [скачать](http://sourceforge.net/projects/editarea/files/) исходники редактора.

Пример простенько страницы:

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<script language="Javascript" type="text/javascript" src="editarea_0_8_2/edit_area/edit_area_full.js"></script>
<script language="Javascript" type="text/javascript">
        editAreaLoader.init({
            id:'example_1',
            change_callback:'onEditAreaChanged',
            language:'ru',
            start_highlight:true,
            allow_resize:'y',
            allow_toggle:true,
            word_wrap:true,
            syntax:'html',
            toolbar:'undo, redo, |, select_font, |, syntax_selection, |, change_smooth_selection, highlight, reset_highlight',
            syntax_selection_allow:'css,html,js,php,xml,sql'
        });

        function onEditAreaChanged(id) {
            document.getElementById(id).value = editAreaLoader.getValue(id);
        }
    </script>
</head>
<body>
<textarea id="example_1" style="height: 150px; width: 100%;" name="test_1">
<?php
    echo 'Hello world!';
?>
</textarea>
</body>
</html>
```
