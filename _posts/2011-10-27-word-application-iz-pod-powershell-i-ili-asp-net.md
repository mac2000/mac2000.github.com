---
layout: post
title: Word.Application из под Powershell и/или Asp.Net

tags: [asp.net, automate, batch, com, dcomcnfg, powershell, word]
---

Если написать простенькую программку или скрипт которая будет что либо делать с word'ом или любым другим продуктом из пакета Microsoft Office - все будет работать ровно до тех пор пока мы не попытаемся вызвать труды нашей работы из под asp.net или другого скритпа, или триггера или шедаллера.

Дело в том что все это добро работает нормально только если запущенно вручную из под вашей учетной записи, иначе просто напросто отваливается, по причине того что офис не может работать вне окружения пользователя.

С горем по полам нашел следующие ссылки:

http://theether.net/download/Microsoft/kb/288368.html

http://theether.net/download/Microsoft/kb/288366.html

Идея в следующем - нам необходимо подправить настройки com объекта word'а чтобы он всегда запускался с определенной учетной записью.

Запускаем: **DCOMCNFG.exe**

Службы компонентов\Компьютеры\Мой компьютер\Настройка DCOM\ и тут самое интересное - ищем среди всего этого бардака нужный нам word.

<amp-img src="/images/wp/128.png" alt="screenshot" width="364" height="218"></amp-img>

У меня на машине стоить Microsoft Office 2010 - но ворд нашел как: **Документ Microsoft Word 97-2003**

В англоязычной системе все может быть совсем наоборот, и что еще более важно - названия может не быть вообще - вместо него может быть соотв. guid.

Для того чтобы наверняка вычислить тот ли объект мы редактируем идем в **regedit.exe** и ищем **winword.exe**

<amp-img src="/images/wp/215.png" alt="screenshot" width="774" height="603"></amp-img>

<amp-img src="/images/wp/37.png" alt="screenshot" width="858" height="605"></amp-img>

Ну и естественно в туле нет никаких фильтров либо возможности поиска, так что придеться все перелопачивать вручную.

Теперь необходимо настроить свойства этого объекта.

Первое что необходимо это разрешить доступ к объекту из нужной нам учетной записи (в моем случае это была сетевая служба)

<amp-img src="/images/wp/46.png" alt="screenshot" width="721" height="572"></amp-img>

Далее необходимо настроить запуск объекта из под необходимой нам учетной записи

<amp-img src="/images/wp/55.png" alt="screenshot" width="410" height="559"></amp-img>

Логично что запись должна существовать, так же было бы не плохо под ней залогиниться и запустить ворд чтобы тот в свою очередь проинициализировался и сделал все свои темные дела.

Все, после этих манипуляций можно дергать ворд из asp.net или powershell да в принципе из чего угодно.

Записи в реестре могут жить в таких местах:

```
HKEY_CLASSES_ROOT\AppID\WINWORD.EXE[@AppID]
HKEY_CLASSES_ROOT\Wow6432Node\AppID\WINWORD.EXE[@AppID]
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\AppID\WINWORD.EXE[@AppID]
HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Wow6432Node\AppID\WINWORD.EXE[@AppID]
HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Classes\AppID\WINWORD.EXE[@AppID]
```
