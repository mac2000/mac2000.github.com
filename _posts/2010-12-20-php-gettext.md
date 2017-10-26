---
layout: post
title: php gettext

tags: [gettext, i18n, l10n, php]
---

При создании сайтов, которые в последствии, возможно будет необходимо перевести на другой язык, либо добавить еще пару языков, встает вопрос о том как сделать так, чтобы когда этот момент настанет перевести сайт без особых затруднений.

Тут поможет [gettext](http://php.net/manual/en/book.gettext.php).Для того чтобы эта штука заработала необходимо по всюду на сайте, где есть вывод какого либо текста, окружать текст вот таким кодом: `_()`.

Например если где либо на странице было вот такое:

```php
<h1>Header for page<?php echo $pagetitle?></h1>
```

Необходимо переделать вот так:

```php
<h1><?php echo _('Header for page').' '.$pagetitle?></h1>
```

Эта функция и будет заниматься переводом всех строк в будущем.

Теперь необходимо в корне проекта создать следующие папки:

```
/locale/ru_RU/LC_MESSAGES
```

Теперь можно запускать [Poedit](http://www.poedit.net/), в котором необходимо создать новый каталог, он создаст необходимые PO файлы, которые в нем же можно и перевести.

Пример кода:

```php
<?php
define('LOCALES_FOLDER_PATH', $_SERVER['DOCUMENT_ROOT'].DIRECTORY_SEPARATOR.'gettext'.DIRECTORY_SEPARATOR.'locale');
define('PO_FILE_NAME', 'default');
$locale = 'ru_RU';
putenv('LANG='.$locale);
setlocale (LC_ALL, $locale);
bindtextdomain (PO_FILE_NAME, LOCALES_FOLDER_PATH);
textdomain (PO_FILE_NAME);
bind_textdomain_codeset(PO_FILE_NAME, 'UTF-8');

echo _("Hello world!");
```
