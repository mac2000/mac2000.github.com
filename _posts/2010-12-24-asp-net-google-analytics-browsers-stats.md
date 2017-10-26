---
layout: post
title: Asp.net Google Analytics Browsers stats

tags: [.net, analytics, asp.net, c#, chart, gdata, google]
---

Нужна была толковая аналитика по браузерам, analytics такой инфы не дает, пришлось набросать свое.

Стандартный asp.net сайт.

Для работы необходимо подключить: `Google.GData.Analytics`, `Google.GData.Client`, `Google.GData.Extensions`.

http://code.google.com/intl/ru-RU/apis/gdata/articles/dotnet_client_lib.html

**Default.aspx**

```csharp
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Default.aspx.cs" Inherits="browserstats._Default" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>browserstats</title>
    <style type="text/css">
        html,body {padding:0;margin:0;overflow:hidden;}
        form {padding:0;margin:0;}
        #browsers-filter-holder {position:absolute;top:1em;left:1em;background-color:#eee;padding:1em;}
        #browsers-filter-holder a {diplay:block;text-decoration:none;border-bottom:1px dashed #0072bc;color:#0072bc;}
        #browsers-filter-holder {background:#FFFFFF;background: -moz-linear-gradient(top,#FFFFFF,#EEEEEE);background: -webkit-gradient(linear, left top, left bottom, from(#FFFFFF), to(#EEEEEE));filter: progid:DXImageTransform.Microsoft.gradient(enabled='true',startColorstr=#FFFFFF,endColorstr=#EEEEEE,GradientType=0);-moz-border-radius:10px;-webkit-border-radius:10px;border-radius:10px;-moz-box-shadow: 0px 0px 5px #000000;-webkit-box-shadow: 0px 0px 5px #000000;box-shadow: 0px 0px 5px #000000;}
        #browsers-filter-holder {filter:progid:DXImageTransform.Microsoft.Alpha(opacity=50);-moz-opacity: 0.5;-khtml-opacity: 0.5;opacity: 0.5;}
        #browsers-filter-holder:hover {filter:progid:DXImageTransform.Microsoft.Alpha(opacity=100);-moz-opacity: 1;-khtml-opacity: 1;opacity: 1;}
    </style>
    <script type="text/javascript" src="https://www.google.com/jsapi"></script>
    <script type="text/javascript">
        function toggle_filter() {
            var cnt = document.getElementById('browsers-filter-content');
            if (!cnt) return false;

            cnt.style.display = (cnt.style.display == 'none') ? 'block' : 'none';

            return false;
        }
    </script>
</head>
<body>
    <form id="form1" runat="server">
        <div id="browsers-filter-holder">
            <a href="javascript:void(0)" onclick="toggle_filter()">Filter results</a>
            <div id="browsers-filter-content" style="display:none;">
                <asp:CheckBoxList ID="cblBrowsers" runat="server"></asp:CheckBoxList>
                <asp:Button ID="btnSubmit" runat="server" onclick="btnSubmit_Click" Text="Submit" />
            </div>
        </div>
        <div id="chart"></div>

        <script type="text/javascript">
            google.load("visualization", "1", {packages:["corechart"]});
            google.setOnLoadCallback(drawChart);
            function drawChart() {
                var data = new google.visualization.DataTable();

                <asp:Literal ID="ltJsData" runat="server" />

                var windowWidth = window.innerWidth ? window.innerWidth : document.documentElement.clientWidth ? document.documentElement.clientWidth : document.body.clientWidth;
                var windowHeight = window.innerHeight ? window.innerHeight : document.documentElement.clientHeight ? document.documentElement.clientHeight : document.body.clientHeight;

                var chart = new google.visualization.LineChart(document.getElementById('chart'));
                chart.draw(data, {
                            width: windowWidth,
                            height: windowHeight,
                            title: 'rabota.ua browsers',
                            hAxis: {title: 'Month'},
                            vAxis: {title: '%', minValue: 0, maxValue: 50},
                            lineWidth: 2,
                            pointSize: 7,

                            });
            }
        </script>

    </form>
</body>
</html>
```

**Default.aspx.cs**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using Google.GData.Analytics;
using Google.GData.Client;
using System.Text.RegularExpressions;
using System.Globalization;
using System.Data;
using System.Text;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;

namespace browserstats
{
    public partial class _Default : System.Web.UI.Page
    {
        public static string LOGIN = "LOGIN@gmail.com";
        public static string PASSWORD = "PASSWORD";
        public static string SITE_ID = "ga:2266524";
        public static int MIN_PERCENTS = 0;
        public static string CACHE_KEY = "rabota_ua_browser_stats";
        public static string DATATABLE_OBJ_PATH = HttpContext.Current.Server.MapPath("~/DataTable.obj");

        protected DataTable Get_DataTable(bool flushCache)
        {
            DataTable dt;
            if (flushCache || DateTime.Now.Month != (new FileInfo(DATATABLE_OBJ_PATH)).LastWriteTime.Month)
            {
                dt = Read_Data_From_Analytics();
                Stream stream = File.Open(DATATABLE_OBJ_PATH, FileMode.Create);
                BinaryFormatter bFormatter = new BinaryFormatter();
                bFormatter.Serialize(stream, dt);
                stream.Close();
            }
            else
            {
                Stream stream = File.Open(DATATABLE_OBJ_PATH, FileMode.Open);
                BinaryFormatter bFormatter = new BinaryFormatter();
                dt = (DataTable)bFormatter.Deserialize(stream);
                stream.Close();
            }
            return dt;
        }

        protected DataTable Read_Data_From_Analytics()
        {
            AnalyticsService service = new AnalyticsService("rabota_ua_browser_stats");
            service.setUserCredentials(LOGIN, PASSWORD);

            string start_date = new DateTime(DateTime.Now.AddYears(-1).Year, DateTime.Now.Month, 1).ToString("yyyy-MM-dd");
            string end_date = new DateTime(DateTime.Now.Year, DateTime.Now.Month, 1).ToString("yyyy-MM-dd");
            List<DataFeed> feeds = new List<DataFeed>();
            string queryUri = "https://www.google.com/analytics/feeds/data" +
                                "?start-date=" + start_date +
                                "&end-date=" + end_date +
                                "&dimensions=ga:month,ga:browser,ga:browserVersion" +
                                "&metrics=ga:pageviews" +
                                "&sort=ga:month" +
                                "&max-results=10000" +
                                "&filters=ga:browserVersion!%3D(not%20set);ga:browserVersion!~%5E0;ga:browser%3D%3DFirefox,ga:browser%3D%3DOpera,ga:browser%3D%3DChrome,ga:browser%3D%3DInternet%20Explorer" +
                                "&ids=" + SITE_ID;

            while (queryUri != null)
            {
                DataFeed dataFeed = service.Query(new DataQuery(queryUri));
                feeds.Add(dataFeed);
                queryUri = dataFeed.NextChunk;
            }

            List<slice> slices = new List<slice>();
            foreach (DataFeed dataFeed in feeds)
            {
                foreach (AtomEntry item in dataFeed.Entries)
                {
                    string month = ((DataEntry)item).Dimensions[0].Value;
                    string browser = ((DataEntry)item).Dimensions[1].Value;
                    string browserVersion = ((DataEntry)item).Dimensions[2].Value;
                    string pageViews = ((DataEntry)item).Metrics[0].Value;

                    browserVersion = Regex.Replace(browserVersion, @"(\d+).*", "$1", RegexOptions.IgnoreCase);

                    int bv = int.Parse(browserVersion);

                    if (bv == 0 || bv > 20) continue;

                    double pv = double.Parse(pageViews);

                    slices.Add(new slice(month, browser, bv, pv));
                }
            }
            feeds = default(List<DataFeed>);

            //group results by month, browser and browserVersion and sum pageViews
            var q = from x in slices
                    group x by new { x.month, x.browser, x.browserVersion } into g
                    select new slice(g.Key.month, g.Key.browser, g.Key.browserVersion, g.Sum(s => s.pageViews));

            //group results by month and sum pageViews
            var pageViewsByMonth = from x in slices
                                    group x by x.month into g
                                    select new { month = g.Key, pageViews = g.Sum(s => s.pageViews) };

            //group results by browser and sum pageViews
            var pageViewsByBrowser = from x in slices
                                        group x by new { x.month, x.browser } into g
                                        select new { month = g.Key.month, browser = g.Key.browser, pageViews = g.Sum(s => s.pageViews) };

            //group results by browser and version and sum pageViews
            var pageViewsByBrowserVersion = from x in slices
                                            group x by new { x.month, x.browser, x.browserVersion } into g
                                            select new { month = g.Key.month, browser = string.Format("{0} {1}", g.Key.browser, g.Key.browserVersion), pageViews = g.Sum(s => s.pageViews) };

            //join grouped results for browsers and their versions
            var totalStatsByBrowsers = pageViewsByBrowser.Union(pageViewsByBrowserVersion).OrderBy(x => x.month).ThenByDescending(x => x.pageViews);

            //convert pageviews to percents
            var totalStatsByBrowsersPercentage = from x in totalStatsByBrowsers
                                                    where Math.Round(x.pageViews / ((pageViewsByMonth.First(m => m.month == x.month)).pageViews / 100)) > MIN_PERCENTS
                                                    select new { month = CultureInfo.CurrentCulture.DateTimeFormat.GetMonthName(int.Parse(x.month)), browser = x.browser, percent = Math.Round(x.pageViews / ((pageViewsByMonth.First(m => m.month == x.month)).pageViews / 100)) };

            //grouped month names
            var tm = from x in totalStatsByBrowsersPercentage
                        group x by x.month into g
                        select g.Key;

            //grouped browser names
            var tb = from x in totalStatsByBrowsersPercentage
                        group x by x.browser into g
                        orderby g.Key
                        select g.Key;

            DataTable dt = new DataTable();
            dt.Columns.Add(new DataColumn("Month", typeof(string)));
            foreach (var b in tb)
            {
                dt.Columns.Add(new DataColumn(b, typeof(double)));
            }

            foreach (string m in tm)
            {
                DataRow dr = dt.NewRow();
                dr["Month"] = m;

                foreach (string b in tb)
                {
                    var o = totalStatsByBrowsersPercentage.FirstOrDefault(x => x.month == m && x.browser == b);
                    string p = (o == null) ? "0" : o.percent.ToString();

                    dr[b] = p;
                }

                dt.Rows.Add(dr);
            }

            return dt;
        }

        protected List<string> Build_Browsers_Filter(DataTable dt)
        {
            List<string> browsers = Build_Browsers_List(dt);
            List<string> browsersFilter = new List<string>();
            if (IsPostBack)
            {
                browsersFilter = (from x in cblBrowsers.Items.Cast<ListItem>()
                                    where x.Selected
                                    select x.Value).ToList<string>();
            }
            else {
                //browsersFilter = browsers;
                browsersFilter.Add("Internet Explorer");
                browsersFilter.Add("Internet Explorer 6");
                browsersFilter.Add("Internet Explorer 7");
                browsersFilter.Add("Internet Explorer 8");
                browsersFilter.Add("Firefox");
                browsersFilter.Add("Opera");
                browsersFilter.Add("Chrome");
            }

            return browsersFilter;
        }

        protected List<string> Build_Browsers_List(DataTable dt)
        {
            return (from x in dt.Columns.Cast<DataColumn>()
                    where x.ColumnName != "Month"
                    select x.ColumnName).ToList<string>();
        }

        protected void Page_Load(object sender, EventArgs e)
        {
            DataTable dt;
            if (HttpContext.Current.Cache[CACHE_KEY] == null)
            {
                dt = Get_DataTable(false); //TODO: add ability to flush cache
                HttpContext.Current.Cache[CACHE_KEY] = dt;
            }
            else
            {
                dt = (DataTable)HttpContext.Current.Cache[CACHE_KEY];
            }

            List<string> browsers = Build_Browsers_List(dt);
            List<string> browsersFilter = Build_Browsers_Filter(dt);

            StringBuilder sb = new StringBuilder();

            sb.AppendLine("data.addColumn('string', 'Month');");
            foreach (string b in browsersFilter)
            {
                sb.AppendLine(string.Format("data.addColumn('number', '{0}');", b));
            }

            sb.AppendLine();
            sb.AppendLine("data.addRows(12);");
            sb.AppendLine();

            for(int r = 0; r < dt.Rows.Count; r++)
            {
                sb.AppendLine(string.Format("data.setValue({0}, 0, '{1}');", r, dt.Rows[r]["Month"]));
                for(int i = 0; i < browsersFilter.Count; i++)
                {
                    sb.AppendLine(string.Format("data.setValue({0}, {1}, {2});", r, (i + 1), dt.Rows[r][browsersFilter[i]]));
                }
                sb.AppendLine();
            }

            ltJsData.Text = sb.ToString();

            cblBrowsers.DataSource = browsers;
            cblBrowsers.DataBind();
        }

        protected void btnSubmit_Click(object sender, EventArgs e)
        {

        }
    }

    public class slice
    {
        public slice(string Month, string Browser, int BrowserVersion, double PageViews)
        {
            month = Month;
            browser = Browser;
            browserVersion = BrowserVersion;
            pageViews = PageViews;
        }
        public string month;
        public string browser;
        public int browserVersion;
        public double pageViews;
    }
}
```

Собственно вот как это все выглядит:

[![video](http://img.youtube.com/vi/blqlUJ-KYY4/0.jpg)](http://www.youtube.com/watch?v=blqlUJ-KYY4)
