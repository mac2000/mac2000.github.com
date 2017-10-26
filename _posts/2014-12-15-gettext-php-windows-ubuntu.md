---
layout: post
title: PHP gettext on Windows and Ubuntu
tags: [php, gettext, i18n, l10n, windows, ubuntu, poedit]
---

Short-long note about using gettext for i18n

Native gettext
--------------

**Vagrantfile**

```php
Vagrant.configure("2") do |config|
    config.vm.box = "wildetech/hyper-u1404"

    config.vm.provider :hyperv do |hv|
    config.vm.synced_folder ".", "/vagrant", type: "smb", smb_username: "Alexandr", smb_password: "******"
    end

    config.vm.provision :shell, path: "Provision.sh"
end
```

**Provision.sh**

```sh
#!/usr/bin/env bash

sudo apt-get update
sudo apt-get install -y gettext php5-cli

sudo apt-get install -y language-pack-en-base
sudo apt-get install -y language-pack-de-base
sudo apt-get install -y language-pack-uk-base
sudo apt-get install -y language-pack-ru-base
```

**ubuntu.php**

```php
<?php

$domain = 'messages';

bindtextdomain($domain, __DIR__ . '/locales'); // gettext will look for .mo files at: `./locales/$locale/LC_MESSAGES/$domain.mo`
//bind_textdomain_codeset($domain, 'UTF-8'); // optional if your encoding is utf
//textdomain($domain); // optional if `$domain == 'messages'`

foreach(['en_US', 'de_DE', 'uk_UA', 'ru_RU'] as $locale) {
    putenv('LANGUAGE=' . $locale);
    setlocale(LC_ALL, $locale . '.utf8');

    echo $locale . ': ' . gettext('Hello World!') . PHP_EOL;
}
```

**windows.php**

```php
<?php

// The only way I found to deal with windows is manually set locale at Control Panel \ Region \ Format
// by changin it to `German (Germany)` or `Ukrainian (Ukraine)` you can change output of script
// the only one nice thing is that changes apply immediatelly without reboots

bindtextdomain('messages', __DIR__ . '/locales');
//bind_textdomain_codeset($domain, 'UTF-8'); // seems to be optional if your encoding is utf
//textdomain($domain); // optional if `$domain == 'messages'`

echo gettext('Hello World!') . PHP_EOL;
```


Benefits from using native gettext unlike any other self written internationalization libraries is that is somehow defacto default way to internationalize anything on linux. But the downside - it is pain to get it working especially on windows.


Symfony translate
-----------------

**composer.json**

```json
{
    "require": {
        "symfony/translation": "2.6.*",
        "symfony/config": "2.6.*"
    }
}
```

**symfony_translate.php**

```php
<?php
use Symfony\Component\Translation\Loader\MoFileLoader;
use Symfony\Component\Translation\Translator;

require_once 'vendor/autoload.php';

$domain = 'messages';

foreach(['en_US', 'de_DE', 'uk_UA', 'ru_RU'] as $locale) {

    $translator = new Translator( $locale );
    $translator->addLoader( $domain, new MoFileLoader());
    $translator->addResource( $domain, __DIR__ . '/locales/' . $locale . '/LC_MESSAGES/' . $domain . '.mo', $locale );

    echo $locale . ': ' . $translator->trans('Hello World!') . PHP_EOL;

}
```

This thing works on both Windows and Ubuntu (actually it does not use gettext and instead parses .mo files, so cache should be enabled).

**Important** for PoEdit to be able to extract your strings you must add `trans` keyword to it.

Twig and Symfony Translate
--------------------------

**composer.json**

```json
{
    "require": {
        "twig/twig": "1.16.*",
        "symfony/translation": "2.6.*",
        "symfony/config": "2.6.*",
        "symfony/twig-bridge": "2.6.*"
    }
}
```

**twig.php**

```php
<?php
use Symfony\Bridge\Twig\Extension\TranslationExtension;
use Symfony\Component\Translation\Loader\MoFileLoader;
use Symfony\Component\Translation\MessageSelector;
use Symfony\Component\Translation\Translator;

require_once 'vendor/autoload.php';

$domain = 'messages';
$locale = 'de_DE';

$translator = new Translator( $locale, new MessageSelector(), __DIR__ . '/cache/translator' );
$translator->addLoader( $domain, new MoFileLoader());
$translator->addResource( $domain, __DIR__ . '/locales/' . $locale . '/LC_MESSAGES/' . $domain . '.mo', $locale );

$twig = new Twig_Environment(new Twig_Loader_Filesystem('templates'), ['cache' => __DIR__ . '/cache/twig', 'debug' => false]); // twig templates should be cached, poEdit will be able to extract strings from cache files
// $twig->addExtension(new Twig_Extension_Debug());
$twig->addExtension(new TranslationExtension($translator));

echo $twig->render('index.twig') . PHP_EOL;
```


So unfortunatelly novadays there is no easy way to use native gettext for internationalization, and if you want to use PoEdit to extract all your strings you should use tricks and hacks like caching twig templates to get your string.

For symfony applications there is self hosted analouge of console poedit which shows what strings are translated and what not.


Twig native gettext
-------------------

just for note

**composer.json**

```json
{
    "require": {
        "twig/twig": "1.16.*",
        "twig/extensions": "1.2.*"
    }
}
```

**twig.php**

```php
<?php
require_once 'vendor/autoload.php';

$locale     = 'de_DE';
$domain = 'messages';

putenv('LANGUAGE=' . $locale);
setlocale(LC_ALL, $locale . '.utf8');
bindtextdomain($domain, __DIR__ . '/locales'); // gettext will look for .mo files at: `./locales/$locale/LC_MESSAGES/$domain.mo`
//bind_textdomain_codeset($domain, 'UTF-8'); // optional if your encoding is utf
//textdomain($domain); // optional if `$domain == 'messages'`

$twig = new Twig_Environment(new Twig_Loader_Filesystem('templates'), ['cache' => __DIR__ . '/cache/twig', 'debug' => false]);
//$twig->addExtension(new Twig_Extension_Debug());
$twig->addExtension(new Twig_Extensions_Extension_I18n());
//$twig->addExtension(new Twig_Extensions_Extension_Intl());


echo $twig->render('index.twig');
```

With caching enabled you can extract all `gettext` function cals from twig cache with poedit.


On windows current locale can be detected from command prompt by running:

```
C:\Users\Alexandr>systeminfo | findstr Locale
System Locale:             uk;Ukrainian
Input Locale:              en-us;English (United States)
```

In all examples I have following localization files:

```
locales/
├── de_DE
│   └── LC_MESSAGES
│       ├── messages.mo
│       └── messages.po
├── ru_RU
│   └── LC_MESSAGES
│       ├── messages.mo
│       └── messages.po
└── uk_UA
    └── LC_MESSAGES
        ├── messages.mo
        └── messages.po
```


It seems that all big projects use own self written internationalization library, Wordpress parses .mo files and caches them, Symfony tries to be able to parse anything possible like Zend do, Laraval suggest use simple arrays etc.

There is also way to use ICUs ResourceBundles but after using similar stuff at ASP.Net I can say it really sucks in comparance with gettext + poedit.

