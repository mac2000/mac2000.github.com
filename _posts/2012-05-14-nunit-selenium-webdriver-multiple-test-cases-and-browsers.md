---
layout: post
title: NUnit, Selenium WebDriver multiple Test Cases and Browsers

tags: [category, ChromeDriver, CurrentContext, FirefoxDriver, FullName, GetScreenshot, InternetExplorerDriver, ITakesScreenshot, IWebDriver, NUnit, OpenQA, result, SaveAsFile, screenshot, Selenium, Test, TestCase, TestContext, TestFixture, TestFixtureSetUp, TestFixtureTearDown, transform, WebDriver, xslt]
---

1. Download and install [NUnit](http://nunit.org).
2. Install NUnit Test Adapter (Tools \ Extensions Manager...)
3. Create Unit Test Project
4. Install (Tools \ Library Package Manager \ Manage NuGet Packages for Solution...):
    * Selenium WebDriver
    * Selenium WebDriver Support Classes
    * NUnit

Here is example code that runs on multiple browsers with multiple test cases:

```csharp
using System;
using System.Text.RegularExpressions;
using System.Threading;
using NUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Firefox;
using OpenQA.Selenium.IE;

namespace Example
{
    [TestFixture(typeof(FirefoxDriver))]
    [TestFixture(typeof(InternetExplorerDriver))]
    [TestFixture(typeof(ChromeDriver))]
    public class TestWithMultipleBrowsers<TWebDriver> where TWebDriver : IWebDriver, new()
    {
        #region Setup
        private IWebDriver driver;

        [TestFixtureSetUp]
        public void CreateDriver()
        {
            if (typeof(TWebDriver).Name == "ChromeDriver")
            {
                // http://code.google.com/p/chromedriver/downloads/list
                driver = new ChromeDriver("C:\\chromedriver");
            }
            else
            {
                driver = new TWebDriver();
            }
        }

        [TearDown]
        public void TearDown()
        {
            // Take screen on failure
            if (TestContext.CurrentContext.Result.Status == TestStatus.Failed)
            {
                string fileName = Regex.Replace(TestContext.CurrentContext.Test.FullName, "[^a-z0-9\\-_]+", "_", RegexOptions.IgnoreCase);
                ((ITakesScreenshot)driver).GetScreenshot().SaveAsFile("C:\\Users\\AlexandrM\\Desktop\\" + fileName + ".png", System.Drawing.Imaging.ImageFormat.Png);
            }
        }

        [TestFixtureTearDown]
        public void FixtureTearDown()
        {
            if (driver != null) driver.Quit();
        }
        #endregion

        #region Tests
        [Test]
        [Description("This test will always fail")]
        public void FailTest()
        {
            driver.Navigate().GoToUrl("http://www.google.com/");
            Thread.Sleep(1000);
            Assert.IsTrue(driver.Title.Contains("rabota.ua"));
        }

        [Test]
        [Description("Test google search")]
        [Category("google"), Category("search")]
        [TestCase("jobsearch")]
        [TestCase("employer")]
        public void GoogleTest(string search)
        {

            driver.Navigate().GoToUrl("http://www.google.com/");
            IWebElement query = driver.FindElement(By.Name("q"));
            query.SendKeys(search + Keys.Enter);

            Thread.Sleep(1000);

            Assert.AreEqual(search + " - Поиск в Google", driver.Title);
        }
        #endregion
    }
}
```

Also you can run [tests via console](http://www.nunit.org/index.php?p=consoleCommandLine&r=2.6), like this:

```
"C:\Program Files (x86)\NUnit 2.6\bin\nunit-console" "C:\Users\AlexandrM\Documents\Visual Studio 11\Projects\Example\Example\Example.csproj"  /result:"C:\Users\AlexandrM\Desktop\result.xml"
```

Notice that result will be saved as XML file, which can be transformed with old [Summary.xslt](http://www.nunit.org/docs/2.2.5/files/Summary.xslt) by adding

```xml
<?xml-stylesheet type="text/xsl" href="Summary.xslt"?>
```

right after xml declaration or you can write your own xslt file.

Here is code from nunit.xslt (can not find link where found it):

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:output method='html' indent='yes'/>
<xsl:template match="/">
    <html>
        <xsl:apply-templates/>
    </html>
</xsl:template>
<xsl:template match="test-results">
<head>
<script type="text/javascript">
function ShowHideDetails(contentControlId) {
    var contentControl = document.getElementById(contentControlId);
    if(contentControl) {
        if(contentControl.style.display == 'none') {
            contentControl.style.display = '';
        } else {
            contentControl.style.display = 'none';
        }
    }
}
</script>
</head>
<body>
<div id="uxHeader" style="font-weight:bold;">
Tests run: <xsl:value-of select="@total"/>, Failures: <xsl:value-of select="@failures"/>, Not run: <xsl:value-of select="@not-run"/>, Time: <xsl:value-of select="test-suite/@time"/> seconds
</div>
<xsl:if test="//test-case[@success = 'False']">
<div style="font-weight:bold;margin-top:10px;border-top:1px solid black;font-size:14pt;">
Failed Tests
</div>
<xsl:for-each select="//test-suite">
<xsl:if test="results/test-case[@success = 'False']">
<div style="margin-top:5px;font-weight:bold;"><xsl:value-of select="@name" /></div>
<div style="margin-left:10px;">
<xsl:call-template name="failureTemplate"></xsl:call-template>
</div>
</xsl:if>
</xsl:for-each>
</xsl:if>
<xsl:if test="//test-case[@executed = 'False']">
<div style="font-weight:bold;margin-top:10px;border-top:1px solid black;font-size:14pt;">
Ignored Tests
</div>
<xsl:for-each select="//test-suite">
<xsl:if test="results/test-case[@executed = 'False']">
<div style="margin-top:5px;font-weight:bold;"><xsl:value-of select="@name" /></div>
<div style="margin-left:10px;">
<xsl:call-template name="ignoreTemplate"></xsl:call-template>
</div>
</xsl:if>
</xsl:for-each>
</xsl:if>
<div style="font-weight:bold;margin-top:10px;border-top:1px solid black;font-size:14pt;">
Successful Tests
</div>
<xsl:for-each select="//test-suite">
<xsl:if test="results/test-case[@success = 'True']">
<div style="margin-top:5px;font-weight:bold;"><xsl:value-of select="@name" /></div>
<div style="margin-left:10px;">
<xsl:call-template name="passTemplate"></xsl:call-template>
</div>
</xsl:if>
</xsl:for-each>
</body>
</xsl:template>
<xsl:template match="results/test-case[failure]" name="failureTemplate">
<xsl:for-each select="results/test-case[@success = 'False']">
<span style="font-weight:bold;font-size:12pt;">
<xsl:value-of select="position()"/>)
</span>
<xsl:value-of select="@name"/>
<div style="font-size:10pt;margin-left:25px;">Reason:
<xsl:choose><xsl:when test="string-length(child::node()/message)=0"> [not defined]</xsl:when>
<xsl:otherwise> "<xsl:value-of select="child::node()/message"/>"</xsl:otherwise>
</xsl:choose>
</div>
</xsl:for-each>
</xsl:template>
<xsl:template match="results/test-case[reason]" name="ignoreTemplate">
<xsl:for-each select="results/test-case[@executed = 'False']">
<span style="font-weight:bold;font-size:12pt;">
<xsl:value-of select="position()"/>)
</span>
<xsl:value-of select="@name"/>
<div style="font-size:10pt;margin-left:25px;">Reason:
<xsl:choose><xsl:when test="string-length(child::node()/message)=0"> Ignored </xsl:when>
<xsl:otherwise> "<xsl:value-of select="child::node()/message"/>"</xsl:otherwise>
</xsl:choose>
</div>
</xsl:for-each>
</xsl:template>
<xsl:template match="results[test-case]" name="passTemplate">
<xsl:for-each select="results/test-case[@success = 'True']">
<span style="font-weight:bold;font-size:12pt;">
<xsl:value-of select="position()"/>)
</span>
<xsl:value-of select="@name"/>
<div style="font-size:10pt;margin-left:25px;">Reported: Success
</div>
<xsl:text></xsl:text>
</xsl:for-each>
</xsl:template>
<xsl:template name="Newline"><xsl:text>
</xsl:text></xsl:template>
</xsl:stylesheet>
```

Here is some screen shots, tests also can be runned from VisualStudio and
NUnit GUI.

<amp-img src="/images/wp/133.png" alt="screenshot" width="839" height="565"></amp-img>

<amp-img src="/images/wp/217.png" alt="screenshot" width="760" height="489"></amp-img>

As of making nice reports, here is starter kit:

```xml
<?xml version="1.0"?>
<!-- http://www.bizcoder.com/index.php/2010/02/12/convert-xml-to-json-using-xslt/ -->
<!-- http://xmlplease.com/whitespace -->
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:output method="html"  indent="yes"/>

    <xsl:template match="/">
        <html>
            <head>
                <meta charset="UTF-8" />
                <title>Test Results</title>
                <link rel="stylesheet" href="http://twitter.github.com/bootstrap/assets/css/bootstrap.css" />
                <!--[if lt IE 9]><script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script><![endif]-->
            </head>
            <body>
                <div data-bind="with: test_results">
                    <p>Date: <span data-bind="text: date"></span></p>
                    <p>Tests: <span data-bind="text: success"></span> of <span data-bind="text: total"></span></p>
                </div>

                <script type="text/javascript" src="http://cloud.github.com/downloads/SteveSanderson/knockout/knockout-2.1.0.js"></script>
                <script type="text/javascript" src="https://raw.github.com/SteveSanderson/knockout.mapping/master/build/output/knockout.mapping-latest.js"></script>
                <script type="text/javascript">
                    var data ={<xsl:apply-templates select="*"/>};
                    data.test_results.success = parseInt(data.test_results.total) - parseInt(data.test_results.errors) - parseInt(data.test_results.failures) - parseInt(data.test_results.ignored) - parseInt(data.test_results.inconclusive) - parseInt(data.test_results.invalid) - parseInt(data.test_results.not_run) - parseInt(data.test_results.skipped);

                    //TODO: do something with data here
                    var results = ko.mapping.fromJS(data);

                    ko.applyBindings(results);
                </script>
            </body>
        </html>
    </xsl:template>

    <!-- Object or Element Property-->
    <xsl:template match="*">
        "<xsl:value-of select="translate(name(), '-', '_')"/>" : <xsl:call-template name="Properties"/>
    </xsl:template>

    <!-- Array Element -->
    <xsl:template match="*" mode="ArrayElement">
        <xsl:call-template name="Properties"/>
    </xsl:template>

    <!-- Object Properties -->
    <xsl:template name="Properties">
        <xsl:variable name="childName" select="name(*[1])"/>
        <xsl:choose>
            <xsl:when test="not(*|@*)">"<xsl:value-of select="normalize-space(translate(., '&#x20;&#x9;&#xD;&#xA;', ' '))"/>"</xsl:when>
            <xsl:when test="count(*[name()=$childName]) > 1">{ "<xsl:value-of select="translate($childName, '-', '_')"/>" :[<xsl:apply-templates select="*" mode="ArrayElement"/>] }</xsl:when>
            <xsl:otherwise>{
                <xsl:apply-templates select="@*"/>
                <xsl:apply-templates select="*"/>
    }</xsl:otherwise>
        </xsl:choose>
        <xsl:if test="following-sibling::*">,</xsl:if>
    </xsl:template>

    <!-- Attribute Property -->
    <xsl:template match="@*">"<xsl:value-of select="translate(name(), '-', '_')"/>" : "<xsl:value-of select="translate(., '&quot;', '\')"/>",
    </xsl:template>
</xsl:stylesheet>
```

The main idea is that all xml is converted to knockout json model, so you can make interactive report.
