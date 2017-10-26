---
layout: post
title: PHP Stemming with PHP Morphy
tags: [stemm, morphy, php, mining]
---

Here is short note how [phpmorphy](http://phpmorphy.sourceforge.net/) can be used in composer based projects to get base worm of word (en, ru)

**composer.json**

```json
{
    "repositories": [
        {
            "type": "package",
            "package": {
                "name": "phpmorphy/ru",
                "version": "0.3.7",
                "dist": {
                    "url": "http://sourceforge.net/projects/phpmorphy/files/phpmorphy-dictionaries/0.3.x/ru_RU/morphy-0.3.x-ru_RU-nojo-utf8.zip/download",
                    "type": "zip"
                }
            }
        },
        {
            "type": "package",
            "package": {
                "name": "phpmorphy/en",
                "version": "0.3.7",
                "dist": {
                    "url": "http://sourceforge.net/projects/phpmorphy/files/phpmorphy-dictionaries/0.3.x/en_EN/morphy-0.3.x-en_EN-windows-1250.zip/download",
                    "type": "zip"
                }
            }
        },
        {
            "type": "package",
            "package": {
                "name": "phpmorphy/phpmorphy",
                "version": "0.3.7",
                "dist": {
                    "url": "http://sourceforge.net/projects/phpmorphy/files/phpmorphy/0.3.7/phpmorphy-0.3.7.zip/download",
                    "type": "zip"
                },
                "require": {
                    "phpmorphy/ru": "x",
                    "phpmorphy/en": "x"
                },
                "autoload": {
                    "files": ["src/common.php"]
                }
            }
        }
    ],
    "require": {
        "phpmorphy/phpmorphy": "0.3.x"
    }
}
```

**play.php**

```php
<?php
require_once 'vendor/autoload.php';

class Morphy
{
    /**
        * @var phpMorphy
        */
    private $enMorphy;

    /**
        * @var phpMorphy
        */
    private $ruMorphy;

    public function __construct($storage = PHPMORPHY_STORAGE_FILE)
    {
        $this->ruMorphy = new phpMorphy(
            new phpMorphy_FilesBundle(realpath(PHPMORPHY_DIR . '/../../ru/'), 'rus'),
            array('storage' => $storage));

        $this->enMorphy = new phpMorphy(
            new phpMorphy_FilesBundle(realpath(PHPMORPHY_DIR . '/../../en/'), 'eng'),
            array('storage' => $storage)
        );
    }

    /**
        * @param string $word to get base from
        * @return string
        */
    public function base($word)
    {
        $sanitizedWord = $this->sanitize($word);
        $result = $this->getMorphy($sanitizedWord)->getBaseForm($sanitizedWord);

        return $result ? mb_strtolower(array_shift($result), 'UTF-8') : null;
    }

    /**
        * @param string $word to sanitize
        * @return string
        */
    private function sanitize($word)
    {
        return mb_strtoupper(trim($word), 'UTF-8');
    }

    /**
        * @param string $word
        * @return phpMorphy
        */
    private function getMorphy($word)
    {
        return $this->isRussian($word) ? $this->ruMorphy : $this->enMorphy;
    }

    /**
        * @param string $word to check
        * @return bool
        */
    private function isRussian($word)
    {
        return preg_match('/[А-Я]/u', $word) ? true : false;
    }
}


$morphy = new Morphy();
echo $morphy->base('Киеве'); // киев
echo $morphy->base('Cities'); // city
```


Pretty of php morphy that there is no external dependencies, and it can be executed even on shared hostings.

For more complex projects probably ElasticSearch will be better solution, it can not only stem but also tokenize text and do many more.

Also there is NLTK project in python which can do all this.
