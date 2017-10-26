---
layout: post
title: User agents list

tags: [browser, curl, php, ua, useragent]
---

```php
<?php

//$uas = get_uas();
//echo '<pre><code>'.print_r($uas, true).'</pre>';
$uas = get_uas();
$res = implode("\n", $uas);
file_put_contents("uas.txt", $res);

function get_uas() {
    $uas = array();

    $cfg = array(
        'Internet Explorer' => array(
            'url' => 'http://www.useragentstring.com/pages/Internet%20Explorer/',
            'versions' => array(6,7,8,9)
        ),
        'Firefox' => array(
            'url' => 'http://www.useragentstring.com/pages/Firefox/',
            'versions' => array(3)
        ),
        'Opera' => array(
            'url' => 'http://www.useragentstring.com/pages/Opera/',
            'versions' => array(10,11)
        ),
        'Chrome' => array(
            'url' => 'http://www.useragentstring.com/pages/Chrome/',
            'versions' => array(8,9,10)
        )
    );

    $regexes = array(
        'Internet Explorer' => 'MSIE (%s)',
        'Firefox' => 'Firefox\/(%s)',
        'Opera' => '(Opera |Version\/)(%s)',
        'Chrome' => 'Chrome\/(%s)'
    );

    foreach($cfg as $browser => $params) {
        $html = file_get_contents($params['url']);
        preg_match_all("/<a[^>]+>(.*?)<\/a>/six", $html, $matches);
        foreach($matches[1] as $ua) {
            $is_browser = preg_match('/'.sprintf($regexes[$browser], implode('|', $params['versions'])).'/i', $ua);
            $is_os = preg_match('/WINDOWS (XP|NT 5|NT 6)/i', $ua);
            if($is_browser && $is_os) {
                $uas[] = $ua;
            }
        }
    }

    return $uas;
}
```
