---
layout: post
title: .net gData analytics

tags: [.net, analytics, c#, gdata, javascript]
---

[Python](http://code.google.com/p/gdata-python-client/)

[JavaScript](http://code.google.com/p/gdata-javascript-client/)

[PHP](http://framework.zend.com/download/gdata)

[.NET](http://code.google.com/p/google-gdata/)

Javascript example
------------------

```html
<script type="text/javascript" src="http://www.google.com/jsapi"></script>
<script type="text/javascript">
    google.load('gdata', '2.x', { packages: ['blogger'] });
    google.load('jquery', '1.3.2');

    google.setOnLoadCallback(function () {
        var status = google.accounts.user.getStatus();
        if (status == google.accounts.AuthSubStatus.LOGGING_IN) {
            // User is in the process of logging in, do nothing.
            return;
        } else if (status == google.accounts.AuthSubStatus.LOGGED_OUT) {
            // User is logged out, display the "login" link.
            alert('user not logged');
            google.accounts.user.login('http://www.blogger.com/feeds');
        } else {
            // User is logged in, load the user's data.
            //transitionDiv('divLoggingIn', 'divLoggedIn', loadData);
            alert('user logged');
            var service = new google.gdata.blogger.BloggerService('AuthSubJS Sample');
            var feedUrl = 'http://www.blogger.com/feeds/default/blogs';
            service.getBlogFeed(feedUrl, loadDataHandler, errorHandler);

        }
    });

    function errorHandler(response) {
        alert('ERROR: ' + response.message);
    }

    function loadDataHandler(response) {
        var msg = '';
        var entries = response.feed.getEntries();
        for (var i = 0; i < entries.length; i++) {
            var entry = entries[i];
            var entryTitle = entry.getTitle().getText();
            msg = msg + entryTitle + '\r\n';
        }
        alert(msg);
    }

</script>
```

.NET example
------------

Add reference to needed dlls from folder: `c:\Program Files (x86)\Google\Google Data API SDK\Redist`

```csharp
FeedQuery query = new FeedQuery();
Service service = new Service("cl", "mac_test_1");
service.setUserCredentials("login@gmail.com", "pwd");

query.Uri = new Uri("http://www.google.com/calendar/feeds/marchenko.alexandr@gmail.com/private/full");

AtomFeed calFeed = service.Query(query);

foreach (AtomEntry item in calFeed.Entries) {
    Console.WriteLine(item.Updated.ToString() + " " + item.Title.Text);
}

Console.ReadKey();
```

more [.net](https://docs.google.com/document/d/1SBx8Fdei_Wljx4mNtpiDZLfipYbO76hdL5lDEvH8lgg/edit?hl=ru) examples:

[C# client](http://code.google.com/intl/ru/apis/gdata/client-cs.html)

[Getting Started](http://google-gdata.googlecode.com/svn/docs/folder1/GettingStarted.html)

Allowed Dimensions and Metrics
------------------------------

[gData Reference Dimensions and Metrincs](http://code.google.com/intl/ru/apis/analytics/docs/gdata/gdataReferenceDimensionsMetrics.html)

.NET page views example
-----------------------

Reference to dlls `Client`, `Analytics`, `Extension`

```csharp
using Google.GData.Client;
using Google.GData.Analytics;

string login = "LOGIN@gmail.com";
string password = "PASSWORD";
string siteId = "ga:11797571";

AnalyticsService service = new AnalyticsService("mac_test_2");
service.setUserCredentials(login, password);

string queryUri = "https://www.google.com/analytics/feeds/data" +
                "?start-date=2010-08-18" +
                "&end-date=2010-08-18" +
                "&dimensions=ga:pageTitle,ga:pagePath" +
                "&metrics=ga:pageviews" +
                "&sort=-ga:pageviews" +
                "&max-results=10" +
                "&ids=" + siteId;

DataFeed dataFeed = service.Query(new DataQuery(queryUri));

foreach (AtomEntry item in dataFeed.Entries)
{
        string pageTitle = ((DataEntry)item).Dimensions[0].Value;
        string pagePath = ((DataEntry)item).Dimensions[1].Value;
        string pageviews = ((DataEntry)item).Metrics[0].Value;

        Console.WriteLine(pageTitle);
        Console.WriteLine(pagePath);
        Console.WriteLine(pageviews);
        Console.WriteLine("------------------------------");
}

Console.ReadKey();
```

Zend Gdata Docs
---------------

[Zend gData](http://framework.zend.com/manual/en/zend.gdata.html)

[Client Login and Authentication for Zend gData](http://www.ngoprekweb.com/2006/11/04/clientlogin-authentication-for-zend-gdata/)

