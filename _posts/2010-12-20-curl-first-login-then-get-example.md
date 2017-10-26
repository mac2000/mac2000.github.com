---
layout: post
title: Curl first login then get example
tags: [automate, curl, php, login, authorize, auth]
---

Curl login

```php
<?php
header("Content-Type: text/html; charset=windows-1251");

$cookie="cookie.txt";

$postdata = "login=LOGIN&pass=PASSWORD";
$ch = curl_init();
curl_setopt ($ch, CURLOPT_URL, "http://example.com");

curl_setopt ($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
curl_setopt ($ch, CURLOPT_USERAGENT, "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.6) Gecko/20070725 Firefox/2.0.0.6");
curl_setopt ($ch, CURLOPT_TIMEOUT, 60);
curl_setopt ($ch, CURLOPT_FOLLOWLOCATION, 1);
curl_setopt ($ch, CURLOPT_RETURNTRANSFER, 0);
curl_setopt ($ch, CURLOPT_COOKIEJAR, $cookie);
curl_setopt ($ch, CURLOPT_REFERER, "http://www.marketing.vc/view_markets.php?num=15825");

curl_setopt ($ch, CURLOPT_POSTFIELDS, $postdata);
curl_setopt ($ch, CURLOPT_POST, 1);
$result = curl_exec ($ch);

curl_setopt($ch, CURLOPT_URL, 'http://www.marketing.vc/view_notes.php?num=88193');

// EXECUTE 2nd REQUEST (FILE DOWNLOAD)
$content = curl_exec ($ch);

curl_close($ch);
echo $content;
```
