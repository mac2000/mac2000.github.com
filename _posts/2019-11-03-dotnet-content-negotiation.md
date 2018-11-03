---
layout: post
title: Dotnet Core Content Negotiation
tags: [dotnet, content negotiation, xml, xsl, xslt, json]
---

Having fun with content negotiation and xslt transformations.

Content Negotiation is an ability of backend to respond to the clients request with appropriate data format (e.g. html, json, xml, etc).

Each client sends `Accept` header with requests which is used to determine appropriate serializer.

For example Chrome, by default uses `text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8` this is why in older .Net WebApi you saw XML instead of desired JSON - reason is content negotiation in action.

The worse thing you might have done is to turn in off, remove all serializers and add only json, just to see json in browser.

In modern dotnet core 2.1 guys turned of content negotiation completelly, so if you have something like this:

```csharp
public class Post
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Body { get; set; }
}

[ApiController]
public class DemoController : ControllerBase
{
    [HttpGet]
    [Route(nameof(Posts))]
    public IEnumerable<Post> Posts() => new[] {
        new Post {
            Id = 1,
            Title = "Hello World",
            Body = "Lorem ipsum dot color"
        },
        new Post {
            Id = 2,
            Title = "Post 2",
            Body = "Lorem ipsum dot color"
        }
    };
}
```

And try to run:

```bash
curl -k -i -s -H 'Accept: text/xml' http://localhost:5000/posts
curl -k -i -s -H 'Accept: application/json' http://localhost:5000/posts
```

Both will return:

```json
[{ "id": 1, "title": "Hello World", "body": "Lorem ipsum dot color" }, { "id": 2, "title": "Post 2", "body": "Lorem ipsum dot color" }]
```

To get content negotiation working back you need to make following ajustments in your `Startup.cs`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options =>
    {
        options.RespectBrowserAcceptHeader = true; // default is false
    })
    .AddXmlSerializerFormatters() // does not added by default
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```

and now suddenly

```bash
curl -k -i -s -H 'Accept: text/xml' http://localhost:5000/posts
```

will return:

```xml
<ArrayOfPost xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Post>
    <Id>1</Id>
    <Title>Hello World</Title>
    <Body>Lorem ipsum dot color</Body>
  </Post>
  <Post>
    <Id>2</Id>
    <Title>Post 2</Title>
    <Body>Lorem ipsum dot color</Body>
  </Post>
</ArrayOfPost>
```

which is desired behavior.

## XSLT

Now fun part, in Chrome we see good old XML, but how about make it more human friendly while still leave as clean XML

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options =>
    {
        options.RespectBrowserAcceptHeader = true; // default is false
        // options.OutputFormatters.Add(new XmlSerializerOutputFormatter()); // not enoug
        options.OutputFormatters.Add(new MyXmlSerializerOutputFormatter());
    })
    // .AddXmlSerializerFormatters() // does not added by default - not enoug
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```

So, we are going to customize XML serilizer a little bit to inject XSLT link to the output.

```csharp
public class MyXmlSerializerOutputFormatter : XmlSerializerOutputFormatter
{
    protected override void Serialize(XmlSerializer xmlSerializer, XmlWriter xmlWriter, object value)
    {
        // TODO: add me only if controller has some kind of custom attribute with XSLT file name
        xmlWriter.WriteProcessingInstruction("xml-stylesheet", "type=\"text/xsl\" href=\"template.xsl\"");
        base.Serialize(xmlSerializer, xmlWriter, value);
    }
}
```

With this small addition your output now will be something like this:

```xml
<?xml-stylesheet type="text/xsl" href="template.xsl"?>
<ArrayOfPost xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Post>
    <Id>1</Id>
    <Title>Hello World</Title>
    <Body>Lorem ipsum dot color</Body>
  </Post>
  <Post>
    <Id>2</Id>
    <Title>Post 2</Title>
    <Body>Lorem ipsum dot color</Body>
  </Post>
</ArrayOfPost>
```

and with small xsl like this:

```xsl
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="html" encoding="utf-8" indent="yes" />

  <xsl:template match="/">
    <xsl:text disable-output-escaping='yes'>&lt;!DOCTYPE html&gt;</xsl:text>
    <html>
    <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Demo</title>
    <style>
        html, body {
            padding: 0;
            margin: 0;
            font: normal 18px/1.5 sans-serif;
            background: #ebebeb;
        }

        body {
            margin: 2em auto;
            max-width: 75vw;
        }

        h1, h3 {
            font-weight: normal;
        }

        summary h3 {
            display: inline;
        }
    </style>
    </head>
    <body>
    <h1>Hello World</h1>
    <xsl:for-each select="//Post">
        <details>
            <summary><h3><xsl:value-of select="Title"/></h3></summary>
            <p><xsl:value-of select="Body" /></p>
        </details>
    </xsl:for-each>
    </body>
    </html>
  </xsl:template>

</xsl:stylesheet>
```

you will have nice rendered page:

<amp-img src="/images/dotnet_content_negotiation_xslt.png" alt="dotnet core xslt transformation for xml output" width="600" height="550"></amp-img>

Whole code:

**Program.cs**

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Xml;
using System.Xml.Serialization;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Formatters;
using Microsoft.Extensions.DependencyInjection;

namespace ContentNegotiation
{
    public class Program
    {
        public static void Main(string[] args) => CreateWebHostBuilder(args).Build().Run();

        public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>();
    }

    public class MyXmlSerializerOutputFormatter : XmlSerializerOutputFormatter
    {
        protected override void Serialize(XmlSerializer xmlSerializer, XmlWriter xmlWriter, object value)
        {
            // TODO: add me only if controller has some kind of custom attribute with XSLT file name
            xmlWriter.WriteProcessingInstruction("xml-stylesheet", "type=\"text/xsl\" href=\"template.xsl\"");
            base.Serialize(xmlSerializer, xmlWriter, value);
        }
    }

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc(options =>
            {
                options.RespectBrowserAcceptHeader = true; // default is false
                // options.OutputFormatters.Add(new XmlSerializerOutputFormatter()); // not enoug
                options.OutputFormatters.Add(new MyXmlSerializerOutputFormatter());
            })
            // .AddXmlSerializerFormatters() // does not added by default, but not enoug
            .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
        }

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseMvc();
        }
    }

    public class Post
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Body { get; set; }
    }

    [ApiController]
    public class DemoController : ControllerBase
    {
        // curl -k -i -s -H 'Accept: text/xml' http://localhost:5000/posts
        // curl -k -i -s -H 'Accept: application/json' http://localhost:5000/posts
        [HttpGet]
        [Route(nameof(Posts))]
        public IEnumerable<Post> Posts() => new[] {
            new Post {
                Id = 1,
                Title = "Hello World",
                Body = "Lorem ipsum dot color"
            },
            new Post {
                Id = 2,
                Title = "Post 2",
                Body = "Lorem ipsum dot color"
            }
        };
    }
}
```
