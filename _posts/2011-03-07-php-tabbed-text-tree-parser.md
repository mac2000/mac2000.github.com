---
layout: post
title: Php tabbed text tree parser

tags: [dom, DOMDocument, domelement, lib, list, nodes, parser, php, tree]
---

This code parses tabbed text trees into DomDocument xml tree

```php
<?php

/**
    * Parses tabbed lines to xml tree
    *
    * @author mac
    */
class TabbedLinesTree {
    /**
        * Parses tabbed list to xml tree
        *
        * @param string $text
        * @param string $delimiter
        * @return DomDocument $dom
        */
    public static function parse($text, $delimiter = "\t") {
        $delimiterRE = '/' . preg_quote($delimiter, "/") . '/';
        $lines = explode(PHP_EOL, trim($text));

        $dom = new DomDocument('1.0', 'utf-8');
        $dom->formatOutput = true;
        $root = $dom->createElement('items');
        $dom->appendChild($root);

        $parent = $root;
        $prev_depth = 0;
        $prev_parent = $root;
        $prev_item = NULL;

        foreach($lines as $line) {
            $value = trim($line);

            if (!empty($value)) { // escape empty items

                $depth = preg_match_all($delimiterRE, $line, $matches);

                // if depth is bigger that in previous step then it is child
                if ($depth > $prev_depth) {
                    $prev_depth = $depth;
                    $prev_parent = $parent;
                    $parent = $prev_item;
                // else it is child of previous parent
                } else if ($depth < $prev_depth) {
                    $parent = $prev_parent;
                    $prev_depth = $depth;
                }
                if ($depth == 0) {
                    $parent = $root;
                    $prev_depth = 0;
                    $prev_parent = $root;
                    $prev_item = NULL;
                }

                $item = $dom->createElement('item');
                $item->setAttribute('value', $value);
                $parent->appendChild($item);
                $prev_item = $item;
            }
        }

        return $dom;
    }

}
```

Here is usage example:

```php
<?php
require_once 'TabbedLinesTree.php';

$text = "Line 1
Line 2
    Line 2.1
    Line 2.2

Line 3";

$dom = TabbedLinesTree::parse($text);

echo $dom->saveXML();
/* Will output:

<?xml version="1.0" encoding="utf-8"?>
<items>
    <item value="Line 1"/>
    <item value="Line 2">
    <item value="Line 2.1"/>
    <item value="Line 2.2"/>
    </item>
    <item value="Line 3"/>
</items>
    */
```

Code with tests [TabbedLinesTree.tar](/images/wp/TabbedLinesTree.tar.gz)
