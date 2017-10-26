---
layout: post
title: Zend Auto Loader

tags: [php, zend, set_include_path, path_separator, realpath, get_include_path, application_path, autoloader, registernamespace]
---

Неободимо сделать  так чтобы папка Zend была доступна из `include_path`.

В примерах используют следующее:

```php
set_include_path(implode(PATH_SEPARATOR, array(
    realpath(APPLICATION_PATH . '/../library'),
    get_include_path(),
)));
```

Если используется **xampp** это дело можно пропустить, так как Zend лежит в `C:\xampp\php\PEAR` который виден для php.

TODO: ссылка на доку по обновлению PEAR

Использование автозагрузчика сводиться к следующему:

```php
require_once 'Zend/Loader/Autoloader.php';
$autoloader = Zend_Loader_Autoloader::getInstance();
$autoloader->registerNamespace('Application_');
```

После этого можно загружать любые классы Zend’а, плюс свои классы из
пространства имен Application

Подробнее:

[Zend loader autoloader](http://framework.zend.com/manual/ru/zend.loader.autoloader.html)

**Application.ini**

```ini
;resources.frontController.controllerDirectory = APPLICATION_PATH "/controllers"
resources.moduleController.controllerDirectory = APPLICATION_PATH "/modules"
autoloaderNamespaces.zc = "ZC_"
resources.frontController.plugins.AssetGrabber = "ZC_Controller_Plugin_AssetsGrabber"

;move controllers and views folders to /modules/default/ folder, also we can create, for example admin direcotry, with its own controllers and views

resources.layout.layoutpath = APPLICATION_PATH "/layouts"
resources.layout.layout = default

resources.frontController.plugins.LayoutPicker = "ZC_Controller_Plugin_LayoutPicker"
```

need `/layouts/admin.phtml` and `layouts/default.phtml`

```php
ZF_Controller_Plugin_LayoutPicker extends Zend_Controller_Plugin_Abstract {
    public function preDispatch(Zend_Controller_Request_Abstract $request) {
        Zend_Layout::getMvcInstance()->setLayout($request->getModuleName());
    }
}
```
