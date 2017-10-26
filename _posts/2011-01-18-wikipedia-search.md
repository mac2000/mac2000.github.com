---
layout: post
title: Wikipedia search

tags: [ajax, php, search, seo, service, wiki, wikipedia]
---

Requests wikipedia search results for given word, and return results as json

```php
<?php
header ('Content-type: application/json; charset=utf-8');

$langs2search = array('ru', 'en');

$q = isset($_REQUEST['q']) ? $_REQUEST['q'] : '';

if(empty($q)) return;

$res = array();
$exact = array();
foreach($langs2search as $lang) {
    try {
        $url = 'http://'.$lang.'.wikipedia.org/w/api.php?action=opensearch&format=json&search='.$q;
        //$content = file_get_contents($url);

        $agents[] = "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; WOW64; SLCC1; .NET CLR 2.0.50727; .NET CLR 3.0.04506; Media Center PC 5.0)";
        $agents[] = "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)";
        $agents[] = "Opera/9.63 (Windows NT 6.0; U; ru) Presto/2.1.1";
        $agents[] = "Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US; rv:1.9.0.5) Gecko/2008120122 Firefox/3.0.5";
        $agents[] = "Mozilla/5.0 (X11; U; Linux i686 (x86_64); en-US; rv:1.8.1.18) Gecko/20081203 Firefox/2.0.0.18";
        $agents[] = "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.16) Gecko/20080702 Firefox/2.0.0.16";
        $agents[] = "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_5_6; en-us) AppleWebKit/525.27.1 (KHTML, like Gecko) Version/3.2.1 Safari/525.27.1";

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_HEADER, false);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
        curl_setopt($ch, CURLOPT_USERAGENT, $agents[rand(0, (count($agents) - 1))]);
        $content = curl_exec($ch);
        curl_close($ch);

        if(empty($content)) continue;
    }
    catch(Exception $e) {
        echo $e->getMessaget();
        continue;
    }

    $data = json_decode($content);

    if(count($data) == 0 || count($data[1]) == 0) continue;

    foreach($data[1] as $link) {
        if(strtolower($link) == strtolower($q)) {
            $exact[] = array(
                'title' => $link,
                'lang' => $lang,
                'url' => 'http://'.$lang.'.wikipedia.org/wiki/' . $link
            );
        }
        else {
            if(count($exact) == 0) {
                $res[] = array(
                    'title' => $link,
                    'lang' => $lang,
                    'url' => 'http://'.$lang.'.wikipedia.org/wiki/' . $link
                );
            }
        }
    }

}
asort($exact);
asort($res);

$res = array_merge($exact, $res);
$res = array_slice($res, 0, min(count($exact), 3));

echo json_encode($res);
```

So now from page:

```js
$.get('wikipedia.php?q='+selection, function(data) {
    if(data.length > 0) {
        var html = '<ol>';
        for(var i = 0; i < data.length; i++) {
            html += '<li><a href="'+data[i].url+'" title="'+data[i].url+'">'+data[i].title+' ('+data[i].lang+')</a></li>';
        }
        html += '</ol>';
...
```
