---
layout: post
title: php DOMDocument print tree

tags: [dom, DOMDocument, html, parse, php, recursive, xml]
---

Небольшой скрипт для вывода дерева элементов документа:

```php
<?php
$html = file_get_contents('http://google.com');

$doc = new DOMDocument();
@$doc->loadHTML($html);
$body = $doc->getElementsByTagName('body')->item(0);

printnodes($body);

function printnodes(DOMNode $node, $level = 0) {
    foreach ($node->childNodes as $child) {
        $id = '';
        $class = '';
        if (method_exists($child, 'getAttribute')) {
            $id = $node->getAttribute('id');
            $class = $node->getAttribute('class');
        }
        if(!empty($id)) $id = '#'.$id;
        if(!empty($class)) $class = '.'.$class;
        $attr = '';
        if(!empty($id) || !empty($class)) $attr = ' ['.$id.$class.']';

        for($i = 0; $i < $level; $i++) {
            echo '&nbsp;';
        }
        echo $child->nodeName . $attr . '<br />';
        if($child->hasChildNodes()) printnodes($child, $level + 4);
    }
}
```

на выходе получится что то вроде такого:

```
textarea
div
    div [#ghead]
        div [#gog]
            nobr [#gbar]
                b
                    #text
                #text
                a
                    #text
                #text
                a
                    #text
                #text
                a
                ...
```
