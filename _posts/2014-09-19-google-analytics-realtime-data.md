---
layout: post
title: Google Analytics RealTime Data
tags: [ga, analytics, realtime, activevisitors, php]
---

Task is to monitor google analytics realtime active visitors counter

https://developers.google.com/api-client-library/php/ - php client library

https://packagist.org/packages/google/apiclient - packagist

http://ga-dev-tools.appspot.com/explorer/ - query builder

https://developers.google.com/apis-explorer/#p/analytics/v3/analytics.data.realtime.get  - realtime analytics query builder

https://developers.google.com/analytics/devguides/reporting/realtime/dimsmets/ - realtime metrics and dimensions

https://console.developers.google.com/project - developers console, you need create project here and turn on analytics API. After that you will be able to create your **Service Account** and get your certificate file. Also **do not forget** to add given service account email address to your analytics account, otherwise you will not have access.

And here is working example:

```php
<?php
require_once 'vendor/autoload.php';

$client = new Google_Client();
$client->setApplicationName('RabotaUA'); // your project name on google developers console

$client->setAssertionCredentials(
    new Google_Auth_AssertionCredentials(
        '********************************************@developer.gserviceaccount.com', // email you added to GA
        ['https://www.googleapis.com/auth/analytics.readonly'],
        file_get_contents(__DIR__ . '/****************************************-privatekey.p12') // keyfile you downloaded
        ));

$client->setClientId('********************************************.apps.googleusercontent.com'); // client id from API console
$client->setAccessType('offline_access'); // this may be unnecessary?

$service = new Google_Service_Analytics($client);

$results = $service->data_realtime->get('ga:*******', 'rt:activeVisitors', ['fields' => 'rows']);

$activeVisitors = $results->rows[0][0];

echo $activeVisitors . PHP_EOL; // 2356
```


'ga:*******' - can be found under Administrator\View settings (3d column)
