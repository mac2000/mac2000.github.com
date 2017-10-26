---
layout: post
title: Seed database with Twitter stream
tags: [seed, twitter, stream, api]
---

While playing with things like replication, sharding etc there is often need for data producer which will continously write some data to database.

Twitter has streeam api which will give you 1+ tweets per second with huge amount of data which can be used for many tests.

Here is short examples:

```csharp
//Program.cs - C# example
//Tweetinvi package required
static void Main(string[] args)
{


    TwitterCredentials.SetCredentials(
        "Access Token",
        "Access Token Secret",
        "Consumer Key (API Key)",
        "Consumer Secret (API Secret)"
    );

    var stream = Stream.CreateSampleStream();

    stream.TweetReceived += (sender, arg) =>
    {
        Console.WriteLine(arg.Tweet.Text); //TODO: write tweet to database
    };

    stream.StartStream();
}
```

and one more:

```php
<?php
require_once 'vendor/autoload.php';

// themattharris/tmhoauth package required

$twitter = new tmhOAuth([
    'consumer_key'    => '',
    'consumer_secret' => '',
    'token'           => '',
    'secret'          => ''
]);

function streaming_callback($data)
{
    $data = json_decode($data, true);
    print_r($data);
}

$twitter->streaming_request(
    'GET',
    'https://stream.twitter.com/1.1/statuses/sample.json',
    [],
    'streaming_callback'
);
```
