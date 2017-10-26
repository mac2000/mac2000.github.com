---
layout: post
title: TortoiseSvn and Jira

tags: [code, jira, scm, svn, tortoisesvn]
---

http://plugins.atlassian.com/plugin/details/10017

Инсталлируем: https://github.com/csharptest/JiraSVN/downloads

Идем в свойства TortoiseSVN

![screenshot](/images/wp/13.png)

Далее добавляем Issue Tracker Integration

![screenshot](/images/wp/2.png)

Где:

* _Working Copy Path_ - путь к снимку репозитория
* _Provide_ - то что мы установили
* _Parameters_ - логин, пароль, адресс jira'ы

http://justaddwater.dk/2010/06/10/windows-svn-pre-commit-hook/

Скрипт который запрещает коммиты без комментария

Создаем файл `D:\SVN\Rabota.UA\hooks\pre-commit.bat` с таким содержимым:

```sh
REM Subversion pre-commit hook for Windows machine
REM put this in your SVN repository folder /hooks/pre-commit.bat
REM we use it with svn version
REM http://stackoverflow.com/questions/869248/windows-pre-commit-hook-for-comment-length-subversion

@echo off
:: Stops commits that have empty log messages.
@echo off

setlocal

rem Subversion sends through the path to the repository and transaction id
set REPOS=%1
set TXN=%2

rem line below ensures at least one character ".", 5 characters require change to "....."
svnlook log %REPOS% -t %TXN% | findstr . > nul
if %errorlevel% gtr 0 (goto err) else exit 0

:err
echo. 1>&2
echo ADD JIRA TASK NUMBER BEFORE COMMIT 1>&2
exit 1
```

Если нужно сделать client side pre-commit hook тогда батник будет выглядеть вот так:

```sh
@echo off
setlocal
set REPOS=%1
set TXN=%2
set COMMENT=%3
rem check that logmessage contains at least 1 characters
type "%COMMENT%" | findstr "." > nul
if %errorlevel% gtr 0 goto err
exit 0
:err
echo Empty log message not allowed. Commit aborted! 1>&2
exit 1
```

Суть изменений в следующем: третьим параметром идет путь к файлу в котором лежит наш коммент, теперь мы проверяем не svnlook'ом, а обычным type'ом, наличие в этом файле текста

Добавляется это дело в настройках TortoiseSVN\Hook Scripts (рядом с Issue Tracking Integration)

http://code.google.com/p/tortoisesvn/source/browse/trunk/contrib/hook-scripts/client-side/PreCommit.js.tmpl?r=19213
