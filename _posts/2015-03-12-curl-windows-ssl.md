---
layout: post
title: Curl SSL on Windows
tags: [curl, ssl, windows, cert]
---

While played with azure found way to get rid of annoing ssl verification erros in curl [here](http://richardwarrender.com/2007/05/the-secret-to-curl-in-php-on-windows/)

All you need to do:

Download [pem](http://curl.haxx.se/ca/cacert.pem) file from http://curl.haxx.se/docs/caextract.html somewhere to local drive

In you code use it like so:

```php
curl_setopt($ch, CURLOPT_CAINFO, "C:/inetpub/wwwroot/cacert.pem");
```

And one more example for guzzle:

```php
$this->client = new Client([
    'base_url' => 'https://testrus.search.windows.net/',
    'defaults' => [
        'verify' => 'C:/inetpub/wwwroot/cacert.pem',
        'headers' => [
            'Content-Type' => 'application/json; charset=utf-8',
            'api-key' => '********************************'
        ],
        'query' => ['api-version' => '2015-02-28-Preview']
    ]
]);
```
