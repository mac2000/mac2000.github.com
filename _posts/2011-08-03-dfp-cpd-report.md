---
layout: post
title: DFP CPD Report

tags: [.net, ad, c#, cpd, dfp, linq, xml, xslt]
---

Double click for publishers does not have report for sponsorship cpd line items, so we need to write own one.

![screenshot](/images/wp/123.png)

First of all we need to get access to Dfp API

http://code.google.com/intl/ru/apis/dfp/docs/start.html

Get production or sandbox access for your needs.

In my project we are using .net so i get .net library here:

http://code.google.com/p/google-api-dfp-dotnet/

**Algo:**

1. Programmaticaly create, run and get results of report by line items.
2. Fetch line items, orders, companies data.
3. Form xml from all retrieved data.
4. Use xslt to make friendly report.

I'm using xml/xslt because you may easily import xml into excel and make some custom analysis.

Something like this:

**Preparations:**

To retrieve data from double click **Google.Dfp.dll** and **System.Web.Services** must be referenced.

Also we need some **App.config** or **Web.config** changes (in my case web).

In **configSections** add:

```xml
<section name="DfpApi" type="System.Configuration.DictionarySectionHandler"/>
```

Also there is need to add **webServices** section into **system.web**:

```xml
<webServices>
  <soapExtensionTypes>
    <add type="Google.Api.Ads.Common.Lib.SoapListenerExtension, Google.Dfp" priority="1" group="0"/>
  </soapExtensionTypes>
</webServices>
```

And add following to your config file:

```xml
<DfpApi>
  <!-- Change the appropriate flags to turn on SOAP logging. -->
  <add key="LogPath" value="C:\dfp\logs"/>
  <add key="LogToConsole" value="false"/>
  <add key="LogToFile" value="false"/>
  <add key="MaskCredentials" value="false"/>
  <add key="LogErrorsOnly" value="true"/>

  <!-- Fill the following values if you plan to use a proxy server.-->
  <add key="ProxyServer" value=""/>
  <add key="ProxyUser" value=""/>
  <add key="ProxyPassword" value=""/>
  <add key="ProxyDomain" value=""/>

  <!-- Use this key to enable or disable gzip compression in SOAP requests.-->
  <add key="EnableGzipCompression" value="true"/>

  <!-- Fill the header values. -->
  <add key="ApplicationName" value="MY_SAMPLE_APP"/>
  <add key="Email" value="LOGIN@gmail.com"/>
  <add key="Password" value="PASSWORD"/>
  <add key="NetworkCode" value="000000"/>

  <!-- Uncomment this if you want to reuse an authToken multiple times. -->
  <!-- <add key="AuthToken" value="ENTER_YOUR_AUTH_TOKEN_HERE"/> -->

  <!-- Uncomment this key if you want to use DFP API sandbox. -->
  <!-- <add key="DfpApi.Server" value="https://sandbox.google.com"/> -->
</DfpApi>
```

Do not forget to change **ApplicationName**, **Email**, **Password** and **NetworkCode** to your values.

**Retrieving data:**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using Google.Api.Ads.Common.Util;
using Google.Api.Ads.Dfp.v201104;
using Google.Api.Ads.Dfp.Lib;
using System.Threading;
using System.Text;
using System.Xml.Linq;
using System.IO;

namespace RabotaUA.Sales
{
    /// <summary>
    /// Summary description for dfpreport
    /// </summary>
    public class dfpreport : IHttpHandler
    {
        public string dfpreportxml = HttpContext.Current.Server.MapPath("~/dfpreport.xml");
        public void ProcessRequest(HttpContext context)
        {
            context.Response.ContentType = "application/xml";
            XDocument doc;
            try
            {
                // if report file is not too old or there is ?refresh=1
                if (File.GetLastWriteTime(dfpreportxml).Month != System.DateTime.Now.Month || !string.IsNullOrEmpty(context.Request.Params["refresh"]))
                {
                    doc = getReport(); // download report from double click
                    doc.Save(dfpreportxml); // save it to file
                }

                context.Response.Write(File.ReadAllText(dfpreportxml)); // write xml response
            }
            catch (Exception ex)
            {
                // if something wrong
                doc = new XDocument(
                        new XDeclaration("1.0", "utf-8", "yes"),
                        new XComment("ERROR"),
                        new XElement("error", ex.Message)
                    );
                context.Response.Write(doc.ToString());
            }

        }

        public XDocument getReport()
        {
            try
            {
                //report start/end dates will be one year
                System.DateTime reportStartDateTime = Utils.Date.getReportStartDateTime();
                System.DateTime reportEndDateTime = Utils.Date.getReportEndDateTime();

                //last month start/end dates
                System.DateTime lastMonthStartDateTime = Utils.Date.getLastMonthStartDateTime();
                System.DateTime lastMonthEndDateTime = Utils.Date.getLastMonthEndDateTime();
                //days in last month
                double days = Utils.Date.getDays(lastMonthStartDateTime, lastMonthEndDateTime);

                CsvFile csv = new CsvFile();
                DfpUser user = new DfpUser();
                ReportService reportService = (ReportService)user.GetService(DfpService.v201104.ReportService);

                #region run report job and wait
                bool reportCompleted = false;
                int totalRetryCount = 0;
                ReportJob reportJob = new ReportJob();
                while (reportCompleted == false)
                {
                    reportJob = Utils.DFP.createReportJob(); // create report job for line items
                    reportJob = reportService.runReportJob(reportJob); // run it
                    int retryCount = 0;

                    while (reportJob.reportJobStatus == ReportJobStatus.IN_PROGRESS) // wait until report completes
                    {
                        Thread.Sleep(3000);
                        if (retryCount++ > 5) break; // if there is timeout
                    }

                    if (reportJob.reportJobStatus == ReportJobStatus.COMPLETED)
                    {
                        reportCompleted = true;
                    }
                    else if (reportJob.reportJobStatus == ReportJobStatus.FAILED)
                    {
                        throw new Exception(string.Format("Report job {0} failed to complete successfully.", reportJob.id));
                    }

                    if (totalRetryCount++ > 2) // some times dpf hangs on and not retrieve report from first time, so we will try again
                    {
                        throw new Exception(string.Format("Report job {0} timed out.", reportJob.id));
                    }
                }
                #endregion

                if (reportCompleted == true) // if all ok, and report job completed successfully
                {
                    #region download report data
                    string url = reportService.getReportDownloadURL(reportJob.id, ExportFormat.CSV);
                    byte[] gzipReport = MediaUtilities.GetAssetDataFromUrl(url);
                    string reportContents = Encoding.UTF8.GetString(MediaUtilities.DeflateGZipData(gzipReport));

                    csv.ReadFromString(reportContents, true);
                    csv.Records.RemoveAt(csv.Records.Count - 1); // remove totals row
                    #endregion

                    List<string> monthNames = csv.Records.Select(x => x[0]).Distinct().ToList(); // get distinct month names
                    string reportMonthName = monthNames.Last(); // get last month name
                    List<string> lineItemIds = csv.Records.Select(x => x[2]).Distinct().ToList(); // get distinct line items ids

                    List<LineItem> lineItems = Utils.DFP.getLineItems(user, csv.Records.Select(x => x[2]).Distinct().ToArray()); // fetch line items, csv.Records.Select(x => x[2]).Distinct() - distinct line item ids
                    List<Order> orders = Utils.DFP.getOrders(user, lineItems.Select(x => x.orderId.ToString()).Distinct().ToArray()); // fetch orders, lineItems.Select(x => x.orderId.ToString()).Distinct() - distinct order ids
                    List<Company> companies = Utils.DFP.getCompanies(user, orders.Select(x => x.advertiserId.ToString()).Distinct().ToArray()); // fetch companies, orders.Select(x => x.advertiserId.ToString()).Distinct() - distinct company ids

                    #region build data
                    // using linq to build data
                    var raw = from item in csv.Records
                              select new
                              {
                                  monthName = item[0],
                                  monthStartDateTime = new System.DateTime(System.DateTime.Now.Year, System.DateTime.Now.Month, 1, 0, 0, 0).AddMonths(-1 * (monthNames.Count - monthNames.IndexOf(item[0]))),
                                  monthEndDateTime = new System.DateTime(System.DateTime.Now.AddMonths(-1 * (monthNames.Count - monthNames.IndexOf(item[0]))).Year, System.DateTime.Now.AddMonths(-1 * (monthNames.Count - monthNames.IndexOf(item[0]))).Month, System.DateTime.DaysInMonth(System.DateTime.Now.AddMonths(-1 * (monthNames.Count - monthNames.IndexOf(item[0]))).Year, System.DateTime.Now.AddMonths(-1 * (monthNames.Count - monthNames.IndexOf(item[0]))).Month), 23, 59, 59),

                                  lineItemName = item[1],
                                  lineItemId = long.Parse(item[2].Replace(".", "").Replace(",", "")),
                                  impressions = long.Parse(item[3].Replace(".", "").Replace(",", "")),
                                  clicks = long.Parse(item[4].Replace(".", "").Replace(",", "")),

                                  orderName = lineItems.Where(x => x.id.ToString() == item[2]).First().orderName,

                                  //orderId = lineItems.Where(li => li.id.ToString() == item[2]).First().orderId,
                                  //advertiserId = orders.Where(o => o.id == lineItems.Where(li => li.id.ToString() == item[2]).First().orderId).First().advertiserId,
                                  companyName = companies.Where(c => c.id == orders.Where(o => o.id == lineItems.Where(li => li.id.ToString() == item[2]).First().orderId).First().advertiserId).First().name,

                                  lineItemStartDateTime = Utils.Date.Convert(lineItems.Where(x => x.id.ToString() == item[2]).First().startDateTime),
                                  lineItemEndDateTime = Utils.Date.Convert(lineItems.Where(x => x.id.ToString() == item[2]).First().endDateTime),

                                  costPerUnit = (long)lineItems.Where(x => x.id.ToString() == item[2]).First().costPerUnit.microAmount / 1000000,
                                  costType = lineItems.Where(x => x.id.ToString() == item[2]).First().costType.ToString(),
                                  status = lineItems.Where(x => x.id.ToString() == item[2]).First().status.ToString(),
                                  budget = (long)lineItems.Where(x => x.id.ToString() == item[2]).First().budget.microAmount / 1000000,
                              };

                    // second step, calculating days and revenues
                    var data = from item in raw
                                select new
                                {
                                    monthName = item.monthName,
                                    monthStartDateTime = item.monthStartDateTime,
                                    monthEndDateTime = item.monthEndDateTime,
                                    lineItemName = item.lineItemName,
                                    lineItemId = item.lineItemId,
                                    impressions = item.impressions,
                                    clicks = item.clicks,
                                    orderName = item.orderName,
                                    companyName = item.companyName,
                                    lineItemStartDateTime = item.lineItemStartDateTime,
                                    lineItemEndDateTime = item.lineItemEndDateTime,
                                    costPerUnit = item.costPerUnit,
                                    costType = item.costType,
                                    budget = item.budget,

                                    monthDays = Utils.Date.getDays(item.monthStartDateTime, item.monthEndDateTime),
                                    lineItemDays = Utils.Date.getDays(item.lineItemStartDateTime, item.lineItemEndDateTime),
                                    lineItemDaysInMonth = Utils.Date.getDaysInRange(item.monthStartDateTime, item.monthEndDateTime, item.lineItemStartDateTime, item.lineItemEndDateTime),

                                    revenueInMonth = item.costPerUnit * Utils.Date.getDaysInRange(item.monthStartDateTime, item.monthEndDateTime, item.lineItemStartDateTime, item.lineItemEndDateTime)
                                };
                    #endregion

                    #region generating xml
                    // create xml from previously formed data
                    XDocument doc = new XDocument(
                        new XDeclaration("1.0", "utf-8", "yes"),
                        new XComment("Report Items"),
                        new XElement("items",
                            from item in data
                            select new XElement("item",
                                new XElement("monthName", item.monthName),
                                new XElement("monthStartDateTime", item.monthStartDateTime),
                                new XElement("monthEndDateTime", item.monthEndDateTime),
                                new XElement("lineItemName", item.lineItemName),
                                new XElement("lineItemId", item.lineItemId),
                                new XElement("impressions", item.impressions),
                                new XElement("clicks", item.clicks),
                                new XElement("orderName", item.orderName),
                                new XElement("companyName", item.companyName),
                                new XElement("lineItemStartDateTime", item.lineItemStartDateTime),
                                new XElement("lineItemEndDateTime", item.lineItemEndDateTime),
                                new XElement("costPerUnit", item.costPerUnit),
                                new XElement("costType", item.costType),
                                new XElement("budget", item.budget),
                                new XElement("monthDays", item.monthDays),
                                new XElement("lineItemDays", item.lineItemDays),
                                new XElement("lineItemDaysInMonth", item.lineItemDaysInMonth),
                                new XElement("revenueInMonth", item.revenueInMonth)
                                )
                            )
                    );

                    // Add the processing instruction.
                    doc.AddFirst(new XProcessingInstruction("xml-stylesheet", "type=\"text/xsl\" href=\"dfpreport.xslt\""));
                    #endregion

                    return doc;
                }

                throw new Exception("Something wrong");

            }
            catch (Exception ex)
            {
                return new XDocument(
                        new XDeclaration("1.0", "utf-8", "yes"),
                        new XComment("ERROR"),
                        new XElement("error", ex.Message)
                    );
            }
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```

Retrieving data may be simpler or more specific, but in my case it is exactly what i want.

**Utils/Date.cs:**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;

namespace RabotaUA.Sales.Utils
{
    public static class Date
    {
        public static System.DateTime getReportStartDateTime()
        {
            return new System.DateTime(System.DateTime.Now.Year, System.DateTime.Now.Month, 1, 0, 0, 0).AddMonths(-1).AddYears(-1);
        }

        public static System.DateTime getReportEndDateTime()
        {
            return new System.DateTime(System.DateTime.Now.AddMonths(-1).Year, System.DateTime.Now.AddMonths(-1).Month, System.DateTime.DaysInMonth(System.DateTime.Now.AddMonths(-1).Year, System.DateTime.Now.AddMonths(-1).Month), 23, 59, 59);
        }

        public static System.DateTime getLastMonthStartDateTime()
        {
            return new System.DateTime(System.DateTime.Now.Year, System.DateTime.Now.Month, 1, 0, 0, 0).AddMonths(-1);
        }

        public static System.DateTime getLastMonthEndDateTime()
        {
            return new System.DateTime(System.DateTime.Now.AddMonths(-1).Year, System.DateTime.Now.AddMonths(-1).Month, System.DateTime.DaysInMonth(System.DateTime.Now.AddMonths(-1).Year, System.DateTime.Now.AddMonths(-1).Month), 23, 59, 59);
        }

        public static System.DateTime Convert(Google.Api.Ads.Dfp.v201104.DateTime date)
        {
            return new System.DateTime(date.date.year, date.date.month, date.date.day, date.hour, date.minute, date.second);
        }

        public static Google.Api.Ads.Dfp.v201104.DateTime Convert(System.DateTime date)
        {
            Google.Api.Ads.Dfp.v201104.DateTime dt = new Google.Api.Ads.Dfp.v201104.DateTime();
            dt.date = new Google.Api.Ads.Dfp.v201104.Date();
            dt.date.year = date.Year;
            dt.date.month = date.Month;
            dt.date.day = date.Day;
            dt.hour = date.Hour;
            dt.minute = date.Minute;
            dt.second = date.Second;

            return dt;
        }

        public static double getDays(System.DateTime startDateTime, System.DateTime endDateTime)
        {
            TimeSpan ts = endDateTime - startDateTime;

            return ts.TotalDays;
        }

        public static double getDaysInRange(System.DateTime startDateTime1, System.DateTime endDateTime1, System.DateTime startDateTime2, System.DateTime endDateTime2)
        {
            System.DateTime s = new System.DateTime(Math.Max(startDateTime2.Ticks, startDateTime1.Ticks));
            System.DateTime e = new System.DateTime(Math.Min(endDateTime2.Ticks, endDateTime1.Ticks));

            TimeSpan ts = e - s;

            if (ts.TotalDays < 0) return 0;

            return ts.TotalDays;
        }
    }
}
```

**Utils/DFP.cs:**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using Google.Api.Ads.Dfp.v201104;
using Google.Api.Ads.Dfp.Lib;

namespace RabotaUA.Sales.Utils
{
    public static class DFP
    {
        public static ReportJob createReportJob()
        {
            System.DateTime reportStartDateTime = Utils.Date.getReportStartDateTime();
            System.DateTime reportEndDateTime = Utils.Date.getReportEndDateTime();

            ReportJob reportJob = new ReportJob();
            reportJob.reportQuery = new ReportQuery();
            reportJob.reportQuery.dimensions = new Dimension[] {
                    Dimension.MONTH,
                    Dimension.LINE_ITEM
                };
            reportJob.reportQuery.columns = new Column[] {
                    Column.AD_SERVER_IMPRESSIONS,
                    Column.AD_SERVER_CLICKS
                };
            reportJob.reportQuery.dateRangeType = DateRangeType.CUSTOM_DATE;

            reportJob.reportQuery.startDate = Utils.Date.Convert(reportStartDateTime).date;
            reportJob.reportQuery.endDate = Utils.Date.Convert(reportEndDateTime).date;

            return reportJob;
        }

        public static List<LineItem> getLineItems(DfpUser user, string[] ids)
        {
            LineItemService service = (LineItemService)user.GetService(DfpService.v201104.LineItemService);

            List<LineItem> items = new List<LineItem>();
            LineItemPage page = new LineItemPage();
            Statement statement = new Statement();
            int offset = 0;
            do
            {
                statement.query = string.Format("WHERE id IN ('{1}') LIMIT 500 OFFSET {0}", offset, string.Join("','", ids));
                page = service.getLineItemsByStatement(statement);

                if (page.results != null && page.results.Length > 0)
                {
                    int i = page.startIndex;
                    foreach (LineItem item in page.results)
                    {
                        items.Add(item);
                        i++;
                    }
                }
                offset += 500;
            } while (offset < page.totalResultSetSize);

            return items;
        }

        public static List<Order> getOrders(DfpUser user, string[] ids)
        {
            OrderService service = (OrderService)user.GetService(DfpService.v201104.OrderService);

            List<Order> items = new List<Order>();
            OrderPage page = new OrderPage();
            Statement statement = new Statement();
            int offset = 0;
            do
            {
                statement.query = string.Format("WHERE id IN ('{1}') LIMIT 500 OFFSET {0}", offset, string.Join("','", ids));
                page = service.getOrdersByStatement(statement);

                if (page.results != null && page.results.Length > 0)
                {
                    int i = page.startIndex;
                    foreach (Order item in page.results)
                    {
                        items.Add(item);
                        i++;
                    }
                }
                offset += 500;
            } while (offset < page.totalResultSetSize);

            return items;
        }

        public static List<Company> getCompanies(DfpUser user, string[] ids)
        {
            CompanyService service = (CompanyService)user.GetService(DfpService.v201104.CompanyService);

            List<Company> items = new List<Company>();
            CompanyPage page = new CompanyPage();
            Statement statement = new Statement();
            int offset = 0;
            do
            {
                statement.query = string.Format("WHERE id IN ('{1}') LIMIT 500 OFFSET {0}", offset, string.Join("','", ids));
                page = service.getCompaniesByStatement(statement);

                if (page.results != null && page.results.Length > 0)
                {
                    int i = page.startIndex;
                    foreach (Company item in page.results)
                    {
                        items.Add(item);
                        i++;
                    }
                }
                offset += 500;
            } while (offset < page.totalResultSetSize);

            return items;
        }
    }
}
```

So at this step we have xml report, like this one:

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<?xml-stylesheet type="text/xsl" href="dfpreport.xslt"?>
<!--Report Items-->
<items>
  <item>
    <monthName>January</monthName>
    <monthStartDateTime>2011-03-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-03-31T23:59:59</monthEndDateTime>
    <lineItemName>test1 premium</lineItemName>
    <lineItemId>3673676</lineItemId>
    <impressions>4</impressions>
    <clicks>0</clicks>
    <orderName>test1</orderName>
    <companyName>RUA Marketing</companyName>
    <lineItemStartDateTime>2011-01-24T10:38:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-01-26T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPM</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>2.55625</lineItemDays>
    <lineItemDaysInMonth>0</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>April</monthName>
    <monthStartDateTime>2011-04-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-04-30T23:59:59</monthEndDateTime>
    <lineItemName>Natasha on search results</lineItemName>
    <lineItemId>4789916</lineItemId>
    <impressions>1118</impressions>
    <clicks>0</clicks>
    <orderName>Test order</orderName>
    <companyName>RUA Marketing</companyName>
    <lineItemStartDateTime>2011-04-21T17:47:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-04-30T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPM</costType>
    <budget>0</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>9.2583333333333329</lineItemDays>
    <lineItemDaysInMonth>9.2583333333333329</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>May</monthName>
    <monthStartDateTime>2011-05-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-05-31T23:59:59</monthEndDateTime>
    <lineItemName>Мarketing (Антощак) marketing_jazzz_468x60_4</lineItemName>
    <lineItemId>5150276</lineItemId>
    <impressions>23401</impressions>
    <clicks>0</clicks>
    <orderName>Marketing Jazzz (Антощак)</orderName>
    <companyName>Marketing_jazzz</companyName>
    <lineItemStartDateTime>2011-05-13T18:06:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-05-30T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPM</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>17.245138888888889</lineItemDays>
    <lineItemDaysInMonth>17.245138888888889</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>8p (Дзюбина) 8p_240x350_1</lineItemName>
    <lineItemId>39463436</lineItemId>
    <impressions>1589</impressions>
    <clicks>0</clicks>
    <orderName>8p (Дзюбина)</orderName>
    <companyName>8p</companyName>
    <lineItemStartDateTime>2011-06-23T18:00:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-08T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>15.249305555555555</lineItemDays>
    <lineItemDaysInMonth>7.2499884259259257</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>8p (Дзюбина) 8p_240x350_2</lineItemName>
    <lineItemId>5566436</lineItemId>
    <impressions>3403</impressions>
    <clicks>16</clicks>
    <orderName>8p (Дзюбина)</orderName>
    <companyName>8p</companyName>
    <lineItemStartDateTime>2011-06-16T12:11:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-08T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>22.491666666666667</lineItemDays>
    <lineItemDaysInMonth>14.492349537037036</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>Epam (Суржак) epam_240x350_tree</lineItemName>
    <lineItemId>5374196</lineItemId>
    <impressions>12934</impressions>
    <clicks>0</clicks>
    <orderName>Epam (Суржак)</orderName>
    <companyName>Epam</companyName>
    <lineItemStartDateTime>2011-05-30T15:10:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-10-31T23:59:00</lineItemEndDateTime>
    <costPerUnit>186</costPerUnit>
    <costType>CPD</costType>
    <budget>28882</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>154.36736111111111</lineItemDays>
    <lineItemDaysInMonth>29.999988425925924</lineItemDaysInMonth>
    <revenueInMonth>5579.9978472222219</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>Epam (Суржак) epam_javaspring_468x60</lineItemName>
    <lineItemId>5360756</lineItemId>
    <impressions>4301</impressions>
    <clicks>0</clicks>
    <orderName>Epam (Суржак)</orderName>
    <companyName>Epam</companyName>
    <lineItemStartDateTime>2011-05-27T15:30:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-06-12T23:59:00</lineItemEndDateTime>
    <costPerUnit>255</costPerUnit>
    <costType>CPD</costType>
    <budget>4347</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>16.353472222222223</lineItemDays>
    <lineItemDaysInMonth>11.999305555555555</lineItemDaysInMonth>
    <revenueInMonth>3059.8229166666665</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>Growthup (Дзюбина)growthup_240x350_3</lineItemName>
    <lineItemId>5413196</lineItemId>
    <impressions>9379</impressions>
    <clicks>42</clicks>
    <orderName>GrowthUp (Дзюбина)</orderName>
    <companyName>GrowthUp</companyName>
    <lineItemStartDateTime>2011-06-01T17:28:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-01T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>30.271527777777777</lineItemDays>
    <lineItemDaysInMonth>29.272210648148146</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>Materialise (Караванов) materialise_dental_160x600_1</lineItemName>
    <lineItemId>5373596</lineItemId>
    <impressions>8705</impressions>
    <clicks>0</clicks>
    <orderName>Materialise (Караванов)</orderName>
    <companyName>Materialise</companyName>
    <lineItemStartDateTime>2011-05-30T14:45:00</lineItemStartDateTime>
    <lineItemEndDateTime>2012-01-01T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>216.38472222222222</lineItemDays>
    <lineItemDaysInMonth>29.999988425925924</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>Materialise (Караванов) materialise_dental_160x600_2</lineItemName>
    <lineItemId>5373476</lineItemId>
    <impressions>8732</impressions>
    <clicks>0</clicks>
    <orderName>Materialise (Караванов)</orderName>
    <companyName>Materialise</companyName>
    <lineItemStartDateTime>2011-05-30T14:43:00</lineItemStartDateTime>
    <lineItemEndDateTime>2012-01-01T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>216.38611111111109</lineItemDays>
    <lineItemDaysInMonth>29.999988425925924</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>June</monthName>
    <monthStartDateTime>2011-06-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-06-30T23:59:59</monthEndDateTime>
    <lineItemName>Тrade_master (Антощак) trade_master240x350_1</lineItemName>
    <lineItemId>39462236</lineItemId>
    <impressions>140</impressions>
    <clicks>0</clicks>
    <orderName>TradeMaster (Антощак)</orderName>
    <companyName>TradeMaster</companyName>
    <lineItemStartDateTime>2011-06-23T12:49:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-08T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>29.999988425925924</monthDays>
    <lineItemDays>15.465277777777777</lineItemDays>
    <lineItemDaysInMonth>7.4659606481481475</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>8p (Дзюбина) 8p_240x350_1</lineItemName>
    <lineItemId>39463436</lineItemId>
    <impressions>1716</impressions>
    <clicks>0</clicks>
    <orderName>8p (Дзюбина)</orderName>
    <companyName>8p</companyName>
    <lineItemStartDateTime>2011-06-23T18:00:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-08T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>15.249305555555555</lineItemDays>
    <lineItemDaysInMonth>7.999305555555555</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>8p (Дзюбина) 8p_240x350_2</lineItemName>
    <lineItemId>5566436</lineItemId>
    <impressions>2179</impressions>
    <clicks>0</clicks>
    <orderName>8p (Дзюбина)</orderName>
    <companyName>8p</companyName>
    <lineItemStartDateTime>2011-06-16T12:11:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-08T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>22.491666666666667</lineItemDays>
    <lineItemDaysInMonth>7.999305555555555</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>Epam (Суржак) epam_240x350_tree</lineItemName>
    <lineItemId>5374196</lineItemId>
    <impressions>16410</impressions>
    <clicks>0</clicks>
    <orderName>Epam (Суржак)</orderName>
    <companyName>Epam</companyName>
    <lineItemStartDateTime>2011-05-30T15:10:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-10-31T23:59:00</lineItemEndDateTime>
    <costPerUnit>186</costPerUnit>
    <costType>CPD</costType>
    <budget>28882</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>154.36736111111111</lineItemDays>
    <lineItemDaysInMonth>30.999988425925924</lineItemDaysInMonth>
    <revenueInMonth>5765.9978472222219</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>Growthup (Дзюбина)growthup_240x350_3</lineItemName>
    <lineItemId>5413196</lineItemId>
    <impressions>256</impressions>
    <clicks>1</clicks>
    <orderName>GrowthUp (Дзюбина)</orderName>
    <companyName>GrowthUp</companyName>
    <lineItemStartDateTime>2011-06-01T17:28:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-01T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>30.271527777777777</lineItemDays>
    <lineItemDaysInMonth>0.99930555555555556</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>Materialise (Караванов) materialise_dental_160x600_1</lineItemName>
    <lineItemId>5373596</lineItemId>
    <impressions>9740</impressions>
    <clicks>0</clicks>
    <orderName>Materialise (Караванов)</orderName>
    <companyName>Materialise</companyName>
    <lineItemStartDateTime>2011-05-30T14:45:00</lineItemStartDateTime>
    <lineItemEndDateTime>2012-01-01T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>216.38472222222222</lineItemDays>
    <lineItemDaysInMonth>30.999988425925924</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>Materialise (Караванов) materialise_dental_160x600_2</lineItemName>
    <lineItemId>5373476</lineItemId>
    <impressions>10209</impressions>
    <clicks>3</clicks>
    <orderName>Materialise (Караванов)</orderName>
    <companyName>Materialise</companyName>
    <lineItemStartDateTime>2011-05-30T14:43:00</lineItemStartDateTime>
    <lineItemEndDateTime>2012-01-01T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>216.38611111111109</lineItemDays>
    <lineItemDaysInMonth>30.999988425925924</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>Scrummaster (Дзюбина) premium banner 240x350</lineItemName>
    <lineItemId>39832076</lineItemId>
    <impressions>1521</impressions>
    <clicks>3</clicks>
    <orderName>Scrummaster (Дзюбина)</orderName>
    <companyName>Scrummaster</companyName>
    <lineItemStartDateTime>2011-07-26T17:56:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-09-08T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>44.252083333333331</lineItemDays>
    <lineItemDaysInMonth>5.2527662037037031</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>Мarketing_jazzz (Антощак) marketing_jazzz_468x60_5</lineItemName>
    <lineItemId>39616796</lineItemId>
    <impressions>8391</impressions>
    <clicks>0</clicks>
    <orderName>Marketing Jazzz (Антощак)</orderName>
    <companyName>Marketing_jazzz</companyName>
    <lineItemStartDateTime>2011-07-06T16:41:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-22T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>16.304166666666667</lineItemDays>
    <lineItemDaysInMonth>16.304166666666667</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
  <item>
    <monthName>July</monthName>
    <monthStartDateTime>2011-07-01T00:00:00</monthStartDateTime>
    <monthEndDateTime>2011-07-31T23:59:59</monthEndDateTime>
    <lineItemName>Тrade_master (Антощак) trade_master240x350_1</lineItemName>
    <lineItemId>39462236</lineItemId>
    <impressions>1754</impressions>
    <clicks>3</clicks>
    <orderName>TradeMaster (Антощак)</orderName>
    <companyName>TradeMaster</companyName>
    <lineItemStartDateTime>2011-06-23T12:49:00</lineItemStartDateTime>
    <lineItemEndDateTime>2011-07-08T23:59:00</lineItemEndDateTime>
    <costPerUnit>0</costPerUnit>
    <costType>CPD</costType>
    <budget>0</budget>
    <monthDays>30.999988425925924</monthDays>
    <lineItemDays>15.465277777777777</lineItemDays>
    <lineItemDaysInMonth>7.999305555555555</lineItemDaysInMonth>
    <revenueInMonth>0</revenueInMonth>
  </item>
</items>
```

Which can be successfully imported into excel to make some analysis.

Next step is make xslt to make from this xml nice report:

**XSLT:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" exclude-result-prefixes="msxsl">
  <xsl:output method="html" indent="yes"/>

  <!-- last item month name -->
  <xsl:variable name="reportMonthName" select="items/item[last()]/monthName" />
  <!-- last item month start date -->
  <xsl:variable name="reportMonthStartDate" select="items/item[last()]/monthStartDateTime" />
  <!-- last item month end date -->
  <xsl:variable name="reportMonthEndDate" select="items/item[last()]/monthEndDateTime" />
  <!-- extract human readable data -->
  <xsl:variable name="reportMonthStartDateString" select="concat(
                      number(substring($reportMonthStartDate, 9, 2)),
                      '/',
                      number(substring($reportMonthStartDate, 6, 2)),
                      '/',
                      number(substring($reportMonthStartDate, 1, 4))
                      )" />
  <!-- extract human readable data -->
  <xsl:variable name="reportMonthEndDateString" select="concat(
                      number(substring($reportMonthEndDate, 9, 2)),
                      '/',
                      number(substring($reportMonthEndDate, 6, 2)),
                      '/',
                      number(substring($reportMonthEndDate, 1, 4))
                      )" />
  <!-- concatenated month start - end human readable dates -->
  <xsl:variable name="reportMonthDatesRangeString" select="concat($reportMonthName, ' (', $reportMonthStartDateString, ' - ', $reportMonthEndDateString, ')')" />

  <!-- concatenated report title -->
  <xsl:variable name="reportTitle" select="concat('Report ', $reportMonthDatesRangeString, '')" />

  <!-- key by month names -->
  <xsl:key name="monthNames" match="item" use="monthName" />
  <!-- key by compnay names -->
  <xsl:key name="companyNames" match="item" use="companyName" />

  <!-- sum all items revenueInMonth in report month -->
  <xsl:variable name="revenueInMonth" select="sum( (key('monthNames', $reportMonthName))/revenueInMonth )"/>

  <!-- line/pie chart width/height -->
  <xsl:variable name="lcw" select="'560'" />
  <xsl:variable name="lch" select="'100'" />
  <xsl:variable name="pcw" select="'560'" />
  <xsl:variable name="pch" select="'200'" />

  <xsl:template match="/">
    <html>
      <head>
        <title>
          <xsl:value-of select="$reportTitle"/>
        </title>
        <style type="text/css">
          html, body {
          font-family: Calibry, Arial;
          font-size:12px;
          color:#36393D;
          background:#fff;
          text-align:center;
          }

          h1 {font-size:24px;font-weight:normal;}
          h2 {font-size:18px;font-weight:normal;}
          h3 {font-size:16px;font-weight:normal;}

          .wrapper {width:560px;text-align:left;margin:auto;}

          table.tbl caption, td, th {padding:2px 5px;}
          table.tbl {border:2px solid #3F4C6B;}
          table.tbl {font-size:12px;}

          table.tbl caption {background:#3F4C6B;color:#fff;}
          table.tbl thead th,
          table.tbl thead td,
          table.tbl tfoot th,
          table.tbl tfoot td {background:#356AA0;color:#fff;font-weight:normal;}
          td.bl {border-left:1px solid #356AA0;}
          table.tbl .even td {background:#C3D9FF}

          /*table.tbl th {text-align:left;}
          table.tbl tfoot th {text-align:right}*/

          td {
          white-space: pre;
          /* CSS 2.0 */
          white-space: pre-wrap;
          /* CSS 2.1 */
          white-space: pre-line;
          /* CSS 3.0 */
          white-space: -pre-wrap;
          /* Opera 4-6 */
          white-space: -o-pre-wrap;
          /* Opera 7 */
          white-space: -moz-pre-wrap;
          /* Mozilla */
          white-space: -hp-pre-wrap;
          /* HP Printers */
          word-wrap: break-word;
          }

        </style>
      </head>
      <body>
        <div class="wrapper">

          <h1>
            <xsl:value-of select="$reportTitle"/>
          </h1>

          <p>
            Revenue in <xsl:value-of select="$reportMonthDatesRangeString"/> is: UAH <b>
              <xsl:value-of select="format-number($revenueInMonth, '0.00')" />
            </b>
          </p>

          <ul>
            <li>
              <a href="#line_items">Line Items</a>
            </li>
            <li>
              <a href="#revenue_by_companies">Revenue By Companies</a>
            </li>

            <li>
              <a href="#companies">Companies</a>
              <ul>
                <li>
                  <a href="#companies_revenue">Revenue</a>
                </li>
                <li>
                  <a href="#companies_avg_cost_per_day">Average Cost Per Day</a>
                </li>
                <li>
                  <a href="#companies_days">Days</a>
                </li>
              </ul>
            </li>

            <li>
              <a href="#monthes">Monthes</a>
              <ul>
                <li>
                  <a href="#monthes_revenue">Revenue</a>
                </li>
                <li>
                  <a href="#monthes_avg_cost_per_day">Average Cost Per Day</a>
                </li>
                <li>
                  <a href="#monthes_impressions">Impressions</a>
                </li>
                <li>
                  <a href="#monthes_clicks">Clicks</a>
                </li>
              </ul>
            </li>
          </ul>

          <a name="line_items"></a>
          <h2>Line Items</h2>
          <xsl:call-template name="line_items_table" />

          <a name="revenue_by_companies"></a>
          <h2>Revenue By Companies</h2>
          <xsl:call-template name="revenue_by_companies_table" />
          <xsl:call-template name="revenue_in_month_by_companies_chart" />

          <a name="companies"></a>
          <h2>Companies</h2>
          <a name="companies_revenue"></a>
          <h3>Revenue</h3>
          <xsl:call-template name="companies_revenue_table" />
          <xsl:call-template name="companies_revenue_chart" />
          <a name="companies_avg_cost_per_day"></a>
          <h3>Average Cost Per Day</h3>
          <xsl:call-template name="companies_avg_cost_per_day_table" />
          <xsl:call-template name="companies_avg_cost_per_day_chart" />
          <a name="companies_days"></a>
          <h3>Days</h3>
          <xsl:call-template name="companies_days_table" />
          <xsl:call-template name="companies_days_chart" />

          <a name="monthes"></a>
          <h2>Monthes</h2>
          <a name="monthes_revenue"></a>
          <h3>Revenue</h3>
          <xsl:call-template name="monthes_revenue_table" />
          <xsl:call-template name="monthes_revenue_chart" />
          <a name="monthes_avg_cost_per_day"></a>
          <h3>Average Cost Per Day</h3>
          <xsl:call-template name="monthes_avg_cost_per_day_table" />
          <xsl:call-template name="monthes_avg_cost_per_day_chart" />
          <a name="monthes_impressions"></a>
          <h3>Impressions</h3>
          <xsl:call-template name="monthes_impressions_table" />
          <xsl:call-template name="monthes_impressions_chart" />
          <a name="monthes_clicks"></a>
          <h3>Clicks</h3>
          <xsl:call-template name="monthes_clicks_table" />
          <xsl:call-template name="monthes_clicks_chart" />

          <!--EXAMPLE GROUP BY MONTH<xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
          <h2>
            <xsl:value-of select="monthName"/>
          </h2>

          <xsl:for-each select="key('monthNames', monthName)">
            <xsl:value-of select="monthName"/> - <xsl:value-of select="orderName"/>
            <br />
          </xsl:for-each>

        </xsl:for-each>-->
          <!--EXAMPLE GROUP BY COMPANY<xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
          <h2>
            <xsl:value-of select="companyName"/>
          </h2>

          <xsl:for-each select="key('companyNames', companyName)">
            <xsl:value-of select="monthName"/> - <xsl:value-of select="orderName"/> - <xsl:value-of select="companyName"/>
            <br />
          </xsl:for-each>

        </xsl:for-each>-->
          <!--EXAMPLE OF INTERSECT<xsl:variable name="a" select="items/item[monthName = 'July']" />
        <xsl:variable name="b" select="items/item[companyName = 'Materialise']" />

        <xsl:for-each select="$a[count(.|$b)=count($b)]">
          <xsl:value-of select="monthName"/> - <xsl:value-of select="companyName"/>
          <br />
        </xsl:for-each>-->

        </div>
      </body>
    </html>
  </xsl:template>

  <xsl:template name="line_items_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Line Items <xsl:value-of select="$reportMonthDatesRangeString"/>
      </caption>
      <thead>
        <th>Line Item Name</th>
        <th class="bl">Budget</th>
        <th class="bl">Impressions</th>
        <th class="bl">Clicks</th>
        <th class="bl">Days In Month</th>
        <th class="bl">Revenue In Month</th>
      </thead>
      <tbody>
        <xsl:for-each select="/items/item[monthName = $reportMonthName]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="lineItemName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number(budget, '0.00')"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="impressions"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="clicks"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number(lineItemDaysInMonth, '0')"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number(revenueInMonth, '0.00')"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Total</th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number( sum( items/item[monthName = $reportMonthName]/budget ) , '0.00')" />
        </th>
        <th class="bl" align="right">
          <xsl:value-of select="sum( items/item[monthName = $reportMonthName]/impressions )" />
        </th>
        <th class="bl" align="right">
          <xsl:value-of select="sum( items/item[monthName = $reportMonthName]/clicks )" />
        </th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number( sum( items/item[monthName = $reportMonthName]/lineItemDaysInMonth ) , '0')" />
        </th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number( sum( items/item[monthName = $reportMonthName]/revenueInMonth ) , '0.00')" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <xsl:template name="revenue_by_companies_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Revenue By Companies <xsl:value-of select="$reportMonthDatesRangeString"  />
      </caption>
      <thead>
        <th>Company</th>
        <th class="bl">Revenue</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="companyName"/>
            </td>
            <td class="bl" align="right">
              <xsl:variable name="by_month" select="key('monthNames', $reportMonthName)" />
              <xsl:variable name="by_company" select="key('companyNames', companyName)" />
              <xsl:variable name="month_revenue_by_company" select="sum( ($by_month[count(.|$by_company)=count($by_company)])/revenueInMonth )" />

              <xsl:value-of select="format-number($month_revenue_by_company, '0.00')"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Total</th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number($revenueInMonth, '0.00')" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <!-- START: COMPANIES TABLES -->

  <xsl:template name="companies_revenue_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Revenue
      </caption>
      <thead>
        <th>Month</th>
        <th class="bl" align="right">Revenue</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="companyName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number( sum( (key('companyNames', companyName))/revenueInMonth )  , '0.00')"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Total</th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number( sum( items/item/revenueInMonth )  , '0.00')" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <xsl:template name="companies_avg_cost_per_day_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Average Cost Per Day
      </caption>
      <thead>
        <th>Company</th>
        <th class="bl" align="right">Average Cost Per Day</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="companyName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number( sum( (key('companyNames', companyName))/costPerUnit ) div count( (key('companyNames', companyName))/costPerUnit ), '0.00')"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Average</th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number( sum( items/item/costPerUnit ) div count( items/item/costPerUnit ) , '0.00')" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <xsl:template name="companies_days_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Days
      </caption>
      <thead>
        <th>Company</th>
        <th class="bl" align="right">Days</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="companyName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number(sum( (key('companyNames', companyName))/lineItemDaysInMonth ), '0')"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Total</th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number(sum( items/item/lineItemDaysInMonth ), '0')" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <!-- END: COMPANIES TABLES -->

  <!-- START: MONTHES TABLES -->

  <xsl:template name="monthes_revenue_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Revenue (<xsl:value-of select="concat(/items/item[1]/monthName, ' - ', /items/item[last()]/monthName)"/>)
      </caption>
      <thead>
        <th>Month</th>
        <th class="bl">Revenue</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="monthName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number( sum( (key('monthNames', monthName))/revenueInMonth ), '0.00')"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Total</th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number( sum( items/item/revenueInMonth ) , '0.00')" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <xsl:template name="monthes_avg_cost_per_day_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Average Cost Per Day (<xsl:value-of select="concat(/items/item[1]/monthName, ' - ', /items/item[last()]/monthName)"/>)
      </caption>
      <thead>
        <th>Month</th>
        <th class="bl" align="right">Average Cost Per Day</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="monthName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="format-number( sum( (key('monthNames', monthName))/costPerUnit ) div count( (key('monthNames', monthName))/costPerUnit ), '0.00')"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Average</th>
        <th class="bl" align="right">
          <xsl:value-of select="format-number( sum( items/item/costPerUnit ) div count( items/item/costPerUnit ) , '0.00')" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <xsl:template name="monthes_impressions_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Impressions (<xsl:value-of select="concat(/items/item[1]/monthName, ' - ', /items/item[last()]/monthName)"/>)
      </caption>
      <thead>
        <th>Month</th>
        <th class="bl" align="right">Impressions</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="monthName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="sum( (key('monthNames', monthName))/impressions )"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Total</th>
        <th class="bl" align="right">
          <xsl:value-of select="sum( items/item/impressions )" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <xsl:template name="monthes_clicks_table">
    <table class="tbl" cellpadding="0" cellspacing="0" border="0" width="100%">
      <caption>
        Clicks (<xsl:value-of select="concat(/items/item[1]/monthName, ' - ', /items/item[last()]/monthName)"/>)
      </caption>
      <thead>
        <th>Month</th>
        <th class="bl" align="right">Clicks</th>
      </thead>
      <tbody>
        <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">

          <xsl:variable name="trClassName">
            <xsl:choose>
              <xsl:when test="position() mod 2 = 1">odd</xsl:when>
              <xsl:otherwise>even</xsl:otherwise>
            </xsl:choose>
          </xsl:variable>

          <tr class="{$trClassName}">
            <td>
              <xsl:value-of select="monthName"/>
            </td>
            <td class="bl" align="right">
              <xsl:value-of select="sum( (key('monthNames', monthName))/clicks )"/>
            </td>
          </tr>
        </xsl:for-each>
      </tbody>
      <tfoot>
        <th>Total</th>
        <th class="bl" align="right">
          <xsl:value-of select="sum( items/item/clicks )" />
        </th>
      </tfoot>
    </table>
  </xsl:template>

  <!-- END: MONTHES TABLES -->

  <!-- START: COMPANIES CHARTS -->

  <xsl:template name="revenue_in_month_by_companies_chart">

    <xsl:variable name="chl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:variable name="by_month" select="key('monthNames', $reportMonthName)" />
        <xsl:variable name="by_company" select="key('companyNames', companyName)" />
        <xsl:variable name="month_revenue_by_company" select="sum( ($by_month[count(.|$by_company)=count($by_company)])/revenueInMonth )" />

        <xsl:value-of select="companyName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:variable name="by_month" select="key('monthNames', $reportMonthName)" />
        <xsl:variable name="by_company" select="key('companyNames', companyName)" />
        <xsl:variable name="month_revenue_by_company" select="sum( ($by_month[count(.|$by_company)=count($by_company)])/revenueInMonth )" />

        <xsl:value-of select="format-number($month_revenue_by_company, '0.00')"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="concat('Revenue In ', $reportMonthDatesRangeString, ' By Companies')" />

    <img src="https://chart.googleapis.com/chart?cht=p3&amp;chs={$pcw}x{$pch}&amp;chd=t:{$chd}&amp;chl={$chl}&amp;chds=a&amp;chco=356AA0" width="{$pcw}" height="{$pch}" alt="{$chtt}" />
  </xsl:template>

  <xsl:template name="companies_revenue_chart">

    <xsl:variable name="chl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:value-of select="companyName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:value-of select="format-number(sum(revenueInMonth), '0')"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="'Revenue'" />

    <img src="https://chart.googleapis.com/chart?cht=p3&amp;chs={$pcw}x{$pch}&amp;chd=t:{$chd}&amp;chl={$chl}&amp;chds=a&amp;chco=356AA0" width="{$pcw}" height="{$pch}" alt="{$chtt}" />
  </xsl:template>

  <xsl:template name="companies_avg_cost_per_day_chart">

    <xsl:variable name="chl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:value-of select="companyName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:value-of select="sum(costPerUnit)"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="'Average Cost Per Day'" />

    <img src="https://chart.googleapis.com/chart?cht=p3&amp;chs={$pcw}x{$pch}&amp;chd=t:{$chd}&amp;chl={$chl}&amp;chds=a&amp;chco=356AA0" width="{$pcw}" height="{$pch}" alt="{$chtt}" />
  </xsl:template>

  <xsl:template name="companies_days_chart">

    <xsl:variable name="chl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:value-of select="companyName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('companyNames', companyName)[1])]">
        <xsl:value-of select="format-number(sum(lineItemDaysInMonth), '0')"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="'Days'" />

    <img src="https://chart.googleapis.com/chart?cht=p3&amp;chs={$pcw}x{$pch}&amp;chd=t:{$chd}&amp;chl={$chl}&amp;chds=a&amp;chco=356AA0" width="{$pcw}" height="{$pch}" alt="{$chtt}" />
  </xsl:template>

  <!-- END: COMPANIES CHARTS -->

  <!-- START: MONTHES CHARTS -->

  <xsl:template name="monthes_revenue_chart">
    <xsl:variable name="chxl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="monthName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="sum( (key('monthNames', monthName))/revenueInMonth )"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="concat('Revenue (', /items/item[1]/monthName, ' - ', /items/item[last()]/monthName, ')')" />

    <img src="https://chart.googleapis.com/chart?cht=lc&amp;chs={$lcw}x{$lch}&amp;chd=t:{$chd}&amp;chxt=x,y&amp;chxl=0:|{$chxl}&amp;chds=a&amp;chco=356AA0" width="{$lcw}" height="{$lch}" alt="{$chtt}" />
  </xsl:template>

  <xsl:template name="monthes_avg_cost_per_day_chart">
    <xsl:variable name="chxl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="monthName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="sum( (key('monthNames', monthName))/costPerUnit ) div count( (key('monthNames', monthName))/costPerUnit )"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="concat('Average Cost Per Day (', /items/item[1]/monthName, ' - ', /items/item[last()]/monthName, ')')" />

    <img src="https://chart.googleapis.com/chart?cht=lc&amp;chs={$lcw}x{$lch}&amp;chd=t:{$chd}&amp;chxt=x,y&amp;chxl=0:|{$chxl}&amp;chds=a&amp;chco=356AA0" width="{$lcw}" height="{$lch}" alt="{$chtt}" />
  </xsl:template>

  <xsl:template name="monthes_impressions_chart">
    <xsl:variable name="chxl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="monthName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="sum( (key('monthNames', monthName))/impressions )"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="concat('Impressions (', /items/item[1]/monthName, ' - ', /items/item[last()]/monthName, ')')" />

    <img src="https://chart.googleapis.com/chart?cht=lc&amp;chs={$lcw}x{$lch}&amp;chd=t:{$chd}&amp;chxt=x,y&amp;chxl=0:|{$chxl}&amp;chds=a&amp;chco=356AA0" width="{$lcw}" height="{$lch}" alt="{$chtt}" />
  </xsl:template>

  <xsl:template name="monthes_clicks_chart">
    <xsl:variable name="chxl">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="monthName"/>
        <xsl:if test="position()!=last()">
          <xsl:text>|</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chd">
      <xsl:for-each select="items/item[generate-id(.) = generate-id(key('monthNames', monthName)[1])]">
        <xsl:value-of select="sum( (key('monthNames', monthName))/clicks )"/>
        <xsl:if test="position()!=last()">
          <xsl:text>,</xsl:text>
        </xsl:if>
      </xsl:for-each>
    </xsl:variable>

    <xsl:variable name="chtt" select="concat('Clicks (', /items/item[1]/monthName, ' - ', /items/item[last()]/monthName, ')')" />

    <img src="https://chart.googleapis.com/chart?cht=lc&amp;chs={$lcw}x{$lch}&amp;chd=t:{$chd}&amp;chxt=x,y&amp;chxl=0:|{$chxl}&amp;chds=a&amp;chco=356AA0" width="{$lcw}" height="{$lch}" alt="{$chtt}" />
  </xsl:template>

  <!-- END: MONTHES CHARTS -->

</xsl:stylesheet>
```

To debug xslt in VisualStudio, define xml file in properties, and press Alt+F5.

At moment working to integrate this report into ExtJs app.
