---
layout: post
title: Keyword phrases

tags: [gdata, google, php, search, seo]
---

Необходимо получить фразу(ы) для ключевого слова, и подменить в них ключевое слово на ссылку с необходимым адресом.

Ф-я получения фраз для ключевого слова:

```php
function getKeywordPhrases($keyword, $target_url) {
    $url_tpl = 'https://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=%s';
    $url = sprintf($url_tpl, urlencode($keyword));
    $results = json_decode(file_get_contents($url));
    $results = $results->responseData->results;
    mb_regex_encoding('utf-8');
    mb_internal_encoding('utf-8');
    $phrases = array();
    foreach($results as $result) {
        $sentences = $result->content;
        $sentences = strip_tags($sentences);
        $sentences = explode(".", $sentences);
        foreach($sentences as $sentence) {
            $sentence = trim($sentence, " ,.:;!?-_()[]\"'`");
            if(empty($sentence)) continue;
            $sentence = mb_strtoupper(mb_substr($sentence, 0, 1, "UTF-8"), "UTF-8").mb_substr($sentence, 1, mb_strlen($sentence), "UTF-8" ) . '.';
            if(!preg_match('/'.$keyword.'/ui', $sentence)) continue;
            $sentence = preg_replace('/'.$keyword.'/ui','<a href="'.$target_url.'" target="_blank">$0</a>', $sentence);
            $phrases[] = $sentence;
        }
    }
    return $phrases;
}
```

И небольшой пример:

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Keyword phrases</title>
<style type="text/css">
html,body {padding:0;margin:0;height:100%;overflow:hidden;background:#DDDDDD;background: -moz-linear-gradient(top,#CCCCCC,#EEEEEE);background: -webkit-gradient(linear, left top, left bottom, from(#CCCCCC), to(#EEEEEE));filter: progid:DXImageTransform.Microsoft.gradient(enabled='true',startColorstr=#CCCCCC,endColorstr=#EEEEEE,GradientType=0);}
form {padding:1em 2em;margin:4em 10em;background:#FFFFFF;background: -moz-linear-gradient(top,#FFFFFF,#EEEEEE);background: -webkit-gradient(linear, left top, left bottom, from(#FFFFFF), to(#EEEEEE));filter: progid:DXImageTransform.Microsoft.gradient(enabled='true',startColorstr=#FFFFFF,endColorstr=#EEEEEE,GradientType=0);-moz-border-radius:10px;-webkit-border-radius:10px;border-radius:10px;-moz-box-shadow: 0px 0px 5px #000000;-webkit-box-shadow: 0px 0px 5px #000000;box-shadow: 0px 0px 5px #000000;}
label {font-weight:bold;}
input {font:14px Arial;padding:0.2em 0.4em;border:1px solid #999;-moz-border-radius:3px;-webkit-border-radius:3px;border-radius:3px;}
button {font:14px Arial;padding:0.2em 0.4em;color:#fff;cursor:pointer;border:none;background-image:-webkit-gradient(linear,left bottom,left top,color-stop(0, rgb(0,0,0)),color-stop(1, rgb(125,126,125)));background-image:-moz-linear-gradient(center bottom,rgb(0,0,0) 0%,rgb(125,126,125) 100%);background-color:#000;-moz-border-radius:3px;-webkit-border-radius:3px;border-radius:3px;}
h3 {margin:0.5em 0;}
</style>
<?php
$keyword = isset($_REQUEST['keyword']) ? $_REQUEST['keyword'] : 'мебель на заказ';
$target_url = isset($_REQUEST['target_url']) ? $_REQUEST['target_url'] : 'http://mebelnazakaz.kiev.ua';
?>
</head>
<body>
    <form>
        <h3>Keyword phrases</h3>
        <label>Keyword:</label>
        <input type="text" name="keyword" value="<?php echo $keyword ?>" />
        <label>Target URL:</label>
        <input type="text" name="target_url" value="<?php echo $target_url ?>" />
        <button>Get phrases</button>
        <h3>Phrases:</h3>
        <ol>
        <?php
            $phrases = getKeywordPhrases($keyword, $target_url);
            foreach($phrases as $phrase) echo "<li>$phrase</li>";
        ?>
        </ol>
    </form>
</body>
</html>
<?php
function getKeywordPhrases($keyword, $target_url) {
    $url_tpl = 'https://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=%s';
    $url = sprintf($url_tpl, urlencode($keyword));
    $results = json_decode(file_get_contents($url));
    $results = $results->responseData->results;
    mb_regex_encoding('utf-8');
    mb_internal_encoding('utf-8');
    $phrases = array();
    foreach($results as $result) {
        $sentences = $result->content;
        $sentences = strip_tags($sentences);
        $sentences = explode(".", $sentences);
        foreach($sentences as $sentence) {
            $sentence = trim($sentence, " ,.:;!?-_()[]\"'`");
            if(empty($sentence)) continue;
            $sentence = mb_strtoupper(mb_substr($sentence, 0, 1, "UTF-8"), "UTF-8").mb_substr($sentence, 1, mb_strlen($sentence), "UTF-8" ) . '.';
            if(!preg_match('/'.$keyword.'/ui', $sentence)) continue;
            $sentence = preg_replace('/'.$keyword.'/ui','<a href="'.$target_url.'" target="_blank">$0</a>', $sentence);
            $phrases[] = $sentence;
        }
    }
    return $phrases;
}
?>
```

Ну и собственно вот как это выглядит:

![screenshot](/images/wp/12.png)

Параметры:

http://code.google.com/intl/ru-RU/apis/websearch/docs/reference.html

Подправленный адрес для более адекватных результатов:

```php
$url_tpl = 'https://ajax.googleapis.com/ajax/services/search/web?v=1.0&hl=ru&rsz=8&q=%s';
```

На последок подправленный вариант, которые идет по всем результатам и дает больше результатов:

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Keyword phrases</title>
<style type="text/css">
html,body {padding:0;margin:0;height:100%;overflow:hidden;background:#DDDDDD;background: -moz-linear-gradient(top,#CCCCCC,#EEEEEE);background: -webkit-gradient(linear, left top, left bottom, from(#CCCCCC), to(#EEEEEE));filter: progid:DXImageTransform.Microsoft.gradient(enabled='true',startColorstr=#CCCCCC,endColorstr=#EEEEEE,GradientType=0);}
form {padding:1em 2em;margin:4em 10em;background:#FFFFFF;background: -moz-linear-gradient(top,#FFFFFF,#EEEEEE);background: -webkit-gradient(linear, left top, left bottom, from(#FFFFFF), to(#EEEEEE));filter: progid:DXImageTransform.Microsoft.gradient(enabled='true',startColorstr=#FFFFFF,endColorstr=#EEEEEE,GradientType=0);-moz-border-radius:10px;-webkit-border-radius:10px;border-radius:10px;-moz-box-shadow: 0px 0px 5px #000000;-webkit-box-shadow: 0px 0px 5px #000000;box-shadow: 0px 0px 5px #000000;}
label {font-weight:bold;}
input {font:14px Arial;padding:0.2em 0.4em;border:1px solid #999;-moz-border-radius:3px;-webkit-border-radius:3px;border-radius:3px;}
button {font:14px Arial;padding:0.2em 0.4em;color:#fff;cursor:pointer;border:none;background-image:-webkit-gradient(linear,left bottom,left top,color-stop(0, rgb(0,0,0)),color-stop(1, rgb(125,126,125)));background-image:-moz-linear-gradient(center bottom,rgb(0,0,0) 0%,rgb(125,126,125) 100%);background-color:#000;-moz-border-radius:3px;-webkit-border-radius:3px;border-radius:3px;}
h3 {margin:0.5em 0;}
</style>
<?php
$keyword = isset($_REQUEST['keyword']) ? $_REQUEST['keyword'] : 'мебель на заказ';
$target_url = isset($_REQUEST['target_url']) ? $_REQUEST['target_url'] : 'http://mebelnazakaz.kiev.ua';
?>
</head>
<body>
    <form>
        <h3>Keyword phrases</h3>
        <label>Keyword:</label>
        <input type="text" name="keyword" value="<?php echo $keyword ?>" />
        <label>Target URL:</label>
        <input type="text" name="target_url" value="<?php echo $target_url ?>" />
        <button>Get phrases</button>
        <h3>Phrases:</h3>
        <ol>
        <?php
            $phrases = getKeywordPhrases($keyword, $target_url);
            foreach($phrases as $phrase) echo "<li>$phrase</li>";
        ?>
        </ol>
    </form>
</body>
</html>
<?php
function getKeywordPhrases($keyword, $target_url) {
    $url_tpl = 'https://ajax.googleapis.com/ajax/services/search/web?v=1.0&start=%d&hl=ru&rsz=8&q=%s';

    $data = array();
    $url = sprintf($url_tpl, 0, urlencode($keyword));
    $results = json_decode(file_get_contents($url));
    $items = $results->responseData->results;
    foreach($items as $item) {
        $data[] = $item->content;
    }
    $pages = $results->responseData->cursor->pages;
    array_shift($pages);
    foreach($pages as $page) {
        $url = sprintf($url_tpl, $page->start, urlencode($keyword));
        $results = json_decode(file_get_contents($url));
        $items = $results->responseData->results;
        foreach($items as $item) {
            $data[] = $item->content;
        }
    }

    mb_regex_encoding('utf-8');
    mb_internal_encoding('utf-8');
    $phrases = array();
    foreach($data as $item) {
        $sentences = strip_tags($item);
        $sentences = explode(".", $sentences);
        foreach($sentences as $sentence) {
            $sentence = trim($sentence, " ,.:;!?-_()[]\"'`");
            if(empty($sentence)) continue;
            $sentence = mb_strtoupper(mb_substr($sentence, 0, 1, "UTF-8"), "UTF-8").mb_substr($sentence, 1, mb_strlen($sentence), "UTF-8" ) . '.';
            if(!preg_match('/'.$keyword.'/ui', $sentence)) continue;
            $sentence = preg_replace('/'.$keyword.'/ui','<a href="'.$target_url.'" target="_blank">$0</a>', $sentence);
            $phrases[] = $sentence;
        }
    }
    return $phrases;
}
?>
```