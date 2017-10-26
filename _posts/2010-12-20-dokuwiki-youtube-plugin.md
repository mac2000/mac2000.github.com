---
layout: post
title: DokuWiki YouTube plugin

tags: [dokuwiki, php, plugin, wiki, youtube, shortcode]
---

Плагин подменяющий ссылки на ютуб на собственно ролики

```php
<?php
/**
    * YouTubeLink Plugin: Converts youtube links
    *
    * @license    GPL 2 (http://www.gnu.org/licenses/gpl.html)
    * @author     Marchenko Alexandr <marchenko.alexandr@gmail.com>
    */

// must be run within Dokuwiki
if(!defined('DOKU_INC')) die();

if(!defined('DOKU_PLUGIN')) define('DOKU_PLUGIN',DOKU_INC.'lib/plugins/');
require_once(DOKU_PLUGIN.'syntax.php');

/**
    * All DokuWiki plugins to extend the parser/rendering mechanism
    * need to inherit from this class
    */
class syntax_plugin_my extends DokuWiki_Syntax_Plugin {
    /**
        * return some info
        */
    function getInfo(){
        return array(
            'author' => 'Marchenko Alexandr',
            'email'  => 'marchenko.alexandr@gmail.com',
            'date'   => '2010-04-16',
            'name'   => 'YouTubeLink Plugin',
            'desc'   => 'Converts youtube links',
            'url'    => 'http://dokuwiki.org/plugin:info',
        );
    }

    /**
        * What kind of syntax are we?
        */
    function getType(){
        return 'substition';
    }

    /**
        * What about paragraphs?
        */
    function getPType(){
        return 'block';
    }

    /**
        * Where to sort in?
        */
    function getSort(){
        return 285;
    }

    /**
        * Connect pattern to lexer
        */
    function connectTo($mode) {
        $this->Lexer->addSpecialPattern('\[\[http:\/\/www\.youtube\.com\/watch\?v=[^\]]+\]\]',$mode,'plugin_my');
    }

    /**
        * Handle the match
        */
    function handle($match, $state, $pos, &$handler) {
        return array($match, $state, $pos);
    }

    /**
        * Create output
        */
    function render($format, &$renderer, $data) {
    if($format == 'xhtml') {
        $v = $data[0];
        $v = trim(str_replace('http://www.youtube.com/watch?v=','',$v),'[]');
            $v = '<object width="560" height="340"><param name="movie" value="http://www.youtube.com/v/'.$v.'&hl=ru_RU&fs=1&"></param><param name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed src="http://www.youtube.com/v/'.$v.'&hl=ru_RU&fs=1&" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="560" height="340"></embed></object>';
        $renderer->doc .= date('r').'<br />'.$v;
            return true;
        }
        return false;
    }
}
```