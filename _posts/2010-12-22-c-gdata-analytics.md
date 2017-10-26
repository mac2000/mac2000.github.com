---
layout: post
title: c# gdata analytics

tags: [.net, analytics, c#, gdata, google]
---

В примере ниже, выдираем инфу за последние 6 месяцев, статистика по браузерам

```csharp
public class slice {
    public slice(string Browser, int BrowserVersion, double PageViews) {
        browser = Browser;
        browserVersion = BrowserVersion;
        pageViews = PageViews;
    }
    public string browser;
    public int browserVersion;
    public double pageViews;
}

public List<List<slice>> getAnalyticsData()
{
    List<List<slice>> slices = new List<List<slice>>();

    string login = "LOGIN@gmail.com";
    string password = "PASSWORD";
    string siteId = "ga:2266524";

    AnalyticsService service = new AnalyticsService("mac_test_3");
    service.setUserCredentials(login, password);

    for (int i = 0; i < 6; i++)
    {
        string start_date = new DateTime(DateTime.Now.Year, DateTime.Now.AddMonths(-6 + i).Month, 1).ToString("yyyy-MM-dd");
        string end_date = new DateTime(DateTime.Now.Year, DateTime.Now.AddMonths(-5 + i).Month, 1).ToString("yyyy-MM-dd");

        string queryUri = "https://www.google.com/analytics/feeds/data" +
                        "?start-date=" + start_date +
                        "&end-date=" + end_date +
                        "&dimensions=ga:browser,ga:browserVersion" +
                        "&metrics=ga:pageviews" +
                        "&sort=-ga:pageviews" +
            //"&max-results=20" +
                        "&filters=ga:browser%3D%3DFirefox,ga:browser%3D%3DOpera,ga:browser%3D%3DChrome,ga:browser%3D%3DInternet Explorer" +
                        "&ids=" + siteId;

        DataFeed dataFeed = service.Query(new DataQuery(queryUri));

        List<slice> list = new List<slice>();

        foreach (AtomEntry item in dataFeed.Entries)
        {
            string browser = ((DataEntry)item).Dimensions[0].Value;
            string browserVersion = ((DataEntry)item).Dimensions[1].Value;
            string pageViews = ((DataEntry)item).Metrics[0].Value;

            if (browserVersion.Contains("(not set)") || browserVersion.StartsWith("999") || browserVersion.StartsWith("0") || browserVersion.StartsWith("99")) continue;

            browserVersion = Regex.Replace(browserVersion, @"(\d+).*", "$1", RegexOptions.IgnoreCase);

            int bv = int.Parse(browserVersion);
            double pv = double.Parse(pageViews);

            list.Add(new slice(browser, bv, pv));
        }

        //group results by browser and browserVersion and sum pageViews
        var q = from x in list
                group x by new { x.browser, x.browserVersion } into g
                select new slice(g.Key.browser, g.Key.browserVersion, g.Sum(s => s.pageViews));

        double totalPageVies = q.Sum(x => x.pageViews);

        //convert pageViews to percents and filter results for min value
        var q2 = from x in q
                    where x.pageViews / (totalPageVies / 100) > 3
                    select new slice(x.browser, x.browserVersion, Math.Round(x.pageViews / (totalPageVies / 100)));

        slices.Add(q2.ToList());
    }

    return slices;
}
```
