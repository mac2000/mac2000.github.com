---
layout: post
title: XSLT Remove Namespace and Move Attributes to Elements
tags: [xslt, xml, namespace]
---

Transforms XML by removing all node and attrbitue prefixes

```xml
<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:output indent="yes" method="xml" encoding="utf-8"/>
    <xsl:strip-space elements="*"/>

    <!-- Stylesheet to remove all namespaces from a document -->
    <!-- NOTE: this will lead to attribute name clash, if an element contains
        two attributes with same local name but different namespace prefix -->
    <!-- Nodes that cannot have a namespace are copied as such -->

    <!-- template to copy elements -->
    <xsl:template match="*">
        <xsl:element name="{local-name()}">
            <xsl:apply-templates select="@* | node()"/>
        </xsl:element>
    </xsl:template>

    <!-- template to copy attributes -->
    <xsl:template match="@*">
        <xsl:attribute name="{local-name()}">
            <xsl:value-of select="."/>
        </xsl:attribute>
    </xsl:template>

    <!-- template to copy the rest of the nodes -->
    <xsl:template match="comment() | text() | processing-instruction()">
        <xsl:copy/>
    </xsl:template>

</xsl:stylesheet>
```

Here sample of iTunes Bodcast feed:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss xmlns:itunes="http://www.itunes.com/dtds/podcast-1.0.dtd" version="2.0">
    <channel>
        <title>All About Everything</title>
        <link>http://www.example.com/podcasts/everything/index.html</link>
        <language>en-us</language>
        <copyright>&#x2117; &amp; &#xA9; 2014 John Doe &amp; Family</copyright>
        <itunes:subtitle>A show about everything</itunes:subtitle>
        <itunes:author>John Doe</itunes:author>
        <itunes:summary>All About Everything is a show about everything. Each week we dive into any subject known to man and talk about it as much as we can. Look for our podcast in the Podcasts app or in the iTunes Store</itunes:summary>
        <description>All About Everything is a show about everything. Each week we dive into any subject known to man and talk about it as much as we can. Look for our podcast in the Podcasts app or in the iTunes Store</description>
        <itunes:owner>
            <itunes:name>John Doe</itunes:name>
            <itunes:email>john.doe@example.com</itunes:email>
        </itunes:owner>
        <itunes:image href="http://example.com/podcasts/everything/AllAboutEverything.jpg"/>
        <itunes:category text="Technology">
            <itunes:category text="Gadgets"/>
        </itunes:category>
        <itunes:category text="TV &amp; Film"/>
        <item>
            <title>Shake Shake Shake Your Spices</title>
            <itunes:author>John Doe</itunes:author>
            <itunes:subtitle>A short primer on table spices</itunes:subtitle>
            <itunes:image href="http://example.com/podcasts/everything/AllAboutEverything/Episode1.jpg"/>
            <enclosure url="http://example.com/podcasts/everything/AllAboutEverythingEpisode3.m4a" length="8727310" type="audio/x-m4a"/>
            <guid>http://example.com/podcasts/archive/aae20140615.m4a</guid>
            <pubDate>Wed, 15 Jun 2014 19:00:00 GMT</pubDate>
            <itunes:duration>7:04</itunes:duration>
        </item>
    </channel>
</rss>
```

And processed version of it:

```xml
<?xml version="1.0" encoding="windows-1251"?>
<rss version="2.0">
    <channel>
        <title>All About Everything</title>
        <link>http://www.example.com/podcasts/everything/index.html</link>
        <language>en-us</language>
        <copyright>&#8471; &amp; © 2014 John Doe &amp; Family</copyright>
        <subtitle>A show about everything</subtitle>
        <author>John Doe</author>
        <summary>All About Everything is a show about everything. Each week we dive into any subject known to man and talk about it as much as we can. Look for our podcast in the Podcasts app or in the iTunes Store</summary>
        <description>All About Everything is a show about everything. Each week we dive into any subject known to man and talk about it as much as we can. Look for our podcast in the Podcasts app or in the iTunes Store</description>
        <owner>
            <name>John Doe</name>
            <email>john.doe@example.com</email>
        </owner>
        <image href="http://example.com/podcasts/everything/AllAboutEverything.jpg"/>
        <category text="Technology">
            <category text="Gadgets"/>
        </category>
        <category text="TV &amp; Film"/>
        <item>
            <title>Shake Shake Shake Your Spices</title>
            <author>John Doe</author>
            <subtitle>A short primer on table spices</subtitle>
            <image href="http://example.com/podcasts/everything/AllAboutEverything/Episode1.jpg"/>
            <enclosure url="http://example.com/podcasts/everything/AllAboutEverythingEpisode3.m4a" length="8727310" type="audio/x-m4a"/>
            <guid>http://example.com/podcasts/archive/aae20140615.m4a</guid>
            <pubDate>Wed, 15 Jun 2014 19:00:00 GMT</pubDate>
            <duration>7:04</duration>
        </item>
    </channel>
</rss>
```

Attributes to Elements
----------------------

```xml
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:output indent="yes" method="xml" encoding="utf-8"/>
    <xsl:strip-space elements="*" />

    <xsl:template match="@* | node()">
        <xsl:copy>
            <xsl:apply-templates select="@* | node()"/>
        </xsl:copy>
    </xsl:template>

    <xsl:template match="@*">
        <xsl:element name="{name()}">
            <xsl:value-of select="."/>
        </xsl:element>
    </xsl:template>

</xsl:stylesheet>
```

such xslt will transform xml like this one:

```xml
<?xml version="1.0" encoding="utf-8"?>
<users>
    <user id="1" blog-url="http://mac-blog.org.ua">
        <firstName>Alexandr</firstName>
        <lastName>Marchenko</lastName>
    </user>
</users>
```

to:

```xml
<?xml version="1.0" encoding="utf-8"?>
<users>
    <user>
        <id>1</id>
        <blog-url>http://mac-blog.org.ua</blog-url>
        <firstName>Alexandr</firstName>
        <lastName>Marchenko</lastName>
    </user>
</users>
```

Both styleshets may be used with something like [jms/serializer](https://packagist.org/packages/jms/serializer) to deserialize almost any kind of xml with almost without effort.
