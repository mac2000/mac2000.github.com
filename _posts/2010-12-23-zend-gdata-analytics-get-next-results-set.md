---
layout: post
title: Zend gdata analytics get next results set

tags: [analytics, gdata, google, php, zend]
---

gdata возвращает по умолчанию до 1000 результатов, можно выставить максимум 10 тыс. но если и этого мало - то надо делать серию запросов с использованием max-results и start-index.

Благо в либе Zend Gdata уже предусмотрели все это дело и добавили метод getNextFeed

Пример:

```php
$email = "LOGIN@gmail.com";
$pass = "PASSWORD";
$siteId = "ga:2266524";

$currentDate = date("Y-m-01");
$startDate = date("Y-m-d", strtotime(date("Y-m-d", strtotime($currentDate)) . " -6 month"));
$endDate = date("Y-m-d", strtotime(date("Y-m-d", strtotime($currentDate)) . " -0 month"));

require_once("Zend/Loader.php");
Zend_Loader::loadClass('Zend_Gdata');
Zend_Loader::loadClass('Zend_Gdata_Query');
Zend_Loader::loadClass('Zend_Gdata_ClientLogin');

$client = Zend_Gdata_ClientLogin::getHttpClient($email, $pass, "analytics");
$gdClient = new Zend_Gdata($client);

$reportURL = "https://www.google.com/analytics/feeds/data" .
    "?start-date=" . $startDate .
    "&end-date=" . $endDate .
    "&dimensions=ga:month,ga:browser,ga:browserVersion" .
    "&metrics=ga:pageviews" .
    "&sort=ga:month" .
    //"&max-results=8" .
    "&filters=ga:browser%3D%3DFirefox,ga:browser%3D%3DOpera,ga:browser%3D%3DChrome,ga:browser%3D%3DInternet%20Explorer" .
    "&ids=" . $siteId;

$results = $gdClient->getFeed($reportURL);
while($results != null) {
    echo count($results).'<br />';
    try {
        $results = $results->getNextFeed();
    }
    catch(Exception $e) {$results = null;}
}
```
