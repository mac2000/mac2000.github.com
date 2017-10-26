---
layout: post
title: Uploadify в ModX

tags: [as3, flash, modx, php, uploadify]
---

В [ModX](http://modxcms.com/) есть полноценный файловый менеджер, который обладает единственным минусом – неудобной загрузкой файлов.

Это становиться настоящей проблемой если создаваемый сайт предполагает необходимость загружать сразу много файлов.

Эту проблему и буду лечить прикрутив [Uploadify](http://www.uploadify.com/), для тех кто не знает, это плагин для [jQuery](http://jquery.com/), позволяющий загружать сразу много файлов, используется в [Wordpress](http://wordpress.org/) по умолчанию.

Для начала необходимо [скачать](http://www.uploadify.com/download/) сам **Uploadify**.

Далее создадим папку `/assets/modules/uploadify` куда и сложим скачанный плагин.

Теперь необходимо найти где в ModX спрятан файловый менеджер и заменить его форму загрузки файлов нашим плагином.

Все что касается файлового менеджера ModX, лежит в файле `/manager/actions/files.dynamic.php`.

В нем в районе **520**-й строки объявляется форма загрузки файлов, которую мы в первую очередь прячем, добавляя стиль `display:none`.

```html
<form enctype="multipart/form-data" action="index.php?a=31" method="post" style="display:none">
```

Далее перед этой формой вставляем следующий код:

```php
<link href="<?php echo $modx->config['base_url']?>assets/modules/uploadify/jquery.uploadify-v2.1.0/uploadify.css" rel="stylesheet" type="text/css" />
<script type="text/javascript" src="<?php echo $modx->config['base_url']?>assets/modules/uploadify/jquery.uploadify-v2.1.0/swfobject.js"></script>
<script type="text/javascript" src="<?php echo $modx->config['base_url']?>assets/js/jquery-1.3.2.min.js"></script>
<script type="text/javascript" src="<?php echo $modx->config['base_url']?>assets/modules/uploadify/jquery.uploadify-v2.1.0/jquery.uploadify.v2.1.0.min.js"></script>
<script type="text/javascript">
var fileCount = 0;
$(document).ready(function() {
    $("#uploadify").uploadify({
        'uploader'       : '<?php echo $modx->config['base_url']?>assets/modules/uploadify/jquery.uploadify-v2.1.0/uploadify.swf',
        'script'         : '<?php echo $modx->config['base_url']?>assets/modules/uploadify/uploadify.php',
        'cancelImg'      : '<?php echo $modx->config['base_url']?>assets/modules/uploadify/jquery.uploadify-v2.1.0/cancel.png',
        'folder'         : '<?php echo $modx->config['base_url']?><?php echo substr($startpath, $len, strlen($startpath))=='' ? '/' : substr($startpath, $len, strlen($startpath))?>',
        'queueID'        : 'fileQueue',
        'auto'           : true,
        'multi'          : true,
                'onSelectOnce'   : function(e, data) { fileCount = data.fileCount; },
                'onAllComplete'  : function(e, data) { if(data.filesUploaded == fileCount) setTimeout('location=location',500); }
    });
});
</script>
<div id="fileQueue"></div><input type="file" name="uploadify" id="uploadify" />
```

Собственно это стандартное включение плагина Uplodify на страницу, не напутайте с путями к файлам Uploadify.

Есть один момент, который стоит оговорить отдельно, дело в том что файловый менеджер ModX аж никак не Ajax’ый, поэтому мы вешаем простенький код на событие `onAllComplete`, которое просто перезагружает страницу, чтобы показать пользователю загруженные файлы.

Осталось реализовать обработку пересылаемых файлов, для этого был создан файл `/assetss/modules/uploadify/uploadify.php` с следующим кодом:

```php
<?php
if (!empty($_FILES)) {
    $tempFile = $_FILES['Filedata']['tmp_name'];
    $targetPath = $_SERVER['DOCUMENT_ROOT'] . $_REQUEST['folder'] . '/';
    $targetFile =  str_replace('//','/',$targetPath) . $_FILES['Filedata']['name'];

    $i = 1;
    while(file_exists($targetFile)) {
        $targetFile = str_replace('//','/',$targetPath) . $i++ . '-' . $_FILES['Filedata']['name'];
    }

    if(move_uploaded_file($tempFile,$targetFile)) {
        echo "1";
        @chmod($targetFile, 0777);
    }
    else {
        echo "Error moving file " . $tempFile . ' to ' . $targetFile;
    }
}
```

Собственно говоря, все. Теперь вместо стандартной формы загрузки файлов, используется Uploadify, который позволяет за раз загружать сколько угодно файлов.
