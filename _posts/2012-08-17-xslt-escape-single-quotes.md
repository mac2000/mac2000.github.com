---
layout: post
title: XSLT escape single quotes
tags: [xml, xsl, xlst, escape, quote, singlequote, call-template, string-length, contains, concat, substring-after]
---

Lets assume that we have some xml and want to convert it into bunch of sql insert queries. The problem is to escape single quotes in them.

Found solution here:

http://codeup.net/2010/05/xslt-recipes-escape-single-quotes/

To call template use code like this:

```xml
<xsl:call-template name="escapeQuotes"><xsl:with-param name="txt" select="name"/></xsl:call-template>
```

This code will return value with escaped quotes.

Here is simple workign example code:

http://mac-blog.org.ua/examples/xslt/escape_single_quotes.xml

escape_single_quotes.xml
------------------------

```xml
<?xml version="1.0" encoding="utf-8"?>
<?xml-stylesheet type="text/xsl" href="escape_single_quotes.xslt"?>
<items>
    <item>
        <name>Alex</name>
        <email>alex@example.com</email>
    </item>
    <item>
        <name>O'reilly</name>
        <email>oreilly@example.com</email>
    </item>
</items>
```

escape_single_quotes.xslt
-------------------------

```xml
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" exclude-result-prefixes="msxsl">
    <xsl:output method="html" indent="yes"/>

    <xsl:template match="/">
        <html>
            <head>
                <title>Escape single quotes example</title>
                <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/twitter-bootstrap/2.0.4/css/bootstrap-combined.min.css" />
            </head>
            <body>
                <div class="container-fluid" style="padding:20px">
                    <xsl:call-template name="table" />
                    <xsl:call-template name="mssql" />
                </div>
            </body>
        </html>
    </xsl:template>

    <xsl:template name="table">
        <table class="table table-striped table-bordered table-condensed">
            <thead>
                <th>Name</th>
                <th>Email</th>
            </thead>
            <tbody>
                <xsl:for-each select="/items/item">
                    <tr>
                        <td><xsl:value-of select="name"/></td>
                        <td><xsl:value-of select="email"/></td>
                    </tr>
                </xsl:for-each>
            </tbody>
        </table>
    </xsl:template>

    <xsl:template name="mssql">
<pre>
<xsl:for-each select="/items/item">INSERT INTO Users VALUES('<xsl:call-template name="escapeQuotes"><xsl:with-param name="txt" select="name"/></xsl:call-template>', '<xsl:value-of select="email"/>');
</xsl:for-each>
</pre>
    </xsl:template>

    <xsl:template name="escapeQuotes">
        <xsl:param name="txt"/>
        <!-- Escape with slash -->
        <!-- <xsl:variable name="backSlashQuote">&#92;&#39;</xsl:variable> -->
        <!-- MsSql escape -->
        <xsl:variable name="backSlashQuote">&#39;&#39;</xsl:variable>
        <xsl:variable name="singleQuote">&#39;</xsl:variable>

        <xsl:choose>
            <xsl:when test="string-length($txt) = 0">
                <!-- ... -->
            </xsl:when>

            <xsl:when test="contains($txt, $singleQuote)">
                <xsl:value-of disable-output-escaping="yes" select="concat(substring-before($txt, $singleQuote), $backSlashQuote)"/>

                <xsl:call-template name="escapeQuotes">
                    <xsl:with-param name="txt" select="substring-after($txt, $singleQuote)"/>
                </xsl:call-template>
            </xsl:when>

            <xsl:otherwise>
                <xsl:value-of disable-output-escaping="yes" select="$txt"/>
            </xsl:otherwise>
        </xsl:choose>
    </xsl:template>

</xsl:stylesheet>
```
