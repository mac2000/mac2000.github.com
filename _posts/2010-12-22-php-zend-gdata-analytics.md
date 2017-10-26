---
layout: post
title: php zend gdata analytics

tags: [analytics, gdata, google, php, service, tool, zend]
---

```php
<?php

$email = "LOGIN@gmail.com";
$pass = "PASSWORD";
$siteId = "ga:2266524";

require_once("Zend/Loader.php");
Zend_Loader::loadClass('Zend_Gdata');
Zend_Loader::loadClass('Zend_Gdata_Query');
Zend_Loader::loadClass('Zend_Gdata_ClientLogin');

$client = Zend_Gdata_ClientLogin::getHttpClient($email, $pass, "analytics");
$gdClient = new Zend_Gdata($client);

try {
    $reportURL = "https://www.google.com/analytics/feeds/data" .
        "?start-date=2010-10-01" .
        "&end-date=2010-11-01" .
        "&dimensions=ga:browser,ga:browserVersion" .
        "&metrics=ga:pageviews" .
        "&sort=-ga:pageviews" .
        //"&max-results=10" .
        "&filters=ga:browser%3D%3DFirefox,ga:browser%3D%3DOpera,ga:browser%3D%3DChrome,ga:browser%3D%3DInternet%20Explorer" .
        "&ids=" . $siteId;

    $results = $gdClient->getFeed($reportURL);

    $data = array();

    foreach($results as $result) {
        $data[] = array(
            'browser' => $result->extensionElements[0]->extensionAttributes['value']['value'],
            'browserVersion' => $result->extensionElements[1]->extensionAttributes['value']['value'],
            'pageViews' => $result->extensionElements[2]->extensionAttributes['value']['value'],
        );
    }

    var_dump($data);
}
catch (Zend_Exception $e) {
    echo "<h3>Caught exception:</h3>" . get_class($e) . "<br />";
    echo "<h3>Message:</h3>" . $e->getMessage() . "<br />";
}
```

В примере выдирается инфа по использованию браузеров, для работы необходима либа Zend Gdata
