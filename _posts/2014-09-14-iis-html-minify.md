---
layout: post
title: IIS Html Minify Module
tags: [iis, minify, module]
---

Unfortunatelly there is not built in IIS functionality to minify HTML output, which is recommended by Google Page Speed Insights.

https://developers.google.com/speed/docs/insights/MinifyResources

There is a way to modify output in module, called "Filters", also there is many examples where Regex used to minify output, but it can be too hard for server on huge amount of clients online.

So, here is simplified implementtion without regex and any other complex data:

```csharp
public override void Write(byte[] buffer, int offset, int count)
{
    char symbol;
    bool betweenTags = false;

    for (long i = 0; i < buffer.Length; i++)
    {
        symbol = Convert.ToChar(buffer[i]);

        if (symbol == '>')
        {
            betweenTags = true;
            responseStream.WriteByte(buffer[i]);
            continue;
        }

        if (betweenTags && Char.IsWhiteSpace(symbol)) continue;

        if (i > 1 && betweenTags && (Convert.ToChar(buffer[i - 1]) == ' ')) responseStream.WriteByte(buffer[i - 1]);

        responseStream.WriteByte(buffer[i]);

        betweenTags = false;
    }
}
```

The idea is as simple as possible, we are going to skip white space symbols between tags.

And here if full code:

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Web;

public class HtmlMinifierModule : IHttpModule
{
    public void Dispose()
    {
    }

    public void Init(HttpApplication context)
    {
        context.ReleaseRequestState += new EventHandler(InstallHtmlMinifierFilter);
    }

    private void InstallHtmlMinifierFilter(object sender, EventArgs e)
    {
        if (HttpContext.Current.Response.ContentType != "text/html") return;

        HttpContext.Current.Response.Filter = new HtmlMinifierFilter(HttpContext.Current.Response.Filter);
    }
}

public class HtmlMinifierFilter : Stream
{
    #region Stream

    private Stream responseStream;
    private long position;

    public HtmlMinifierFilter(Stream inputStream)
    {
        responseStream = inputStream;
    }

    public override bool CanRead
    {
        get
        {
            return true;
        }
    }

    public override bool CanSeek
    {
        get
        {
            return true;
        }
    }

    public override bool CanWrite
    {
        get
        {
            return true;
        }
    }

    public override void Flush()
    {
        responseStream.Flush();
    }

    public override long Length
    {
        get
        {
            return 0;
        }
    }

    public override long Position
    {
        get
        {
            return position;
        }
        set
        {
            position = value;
        }
    }

    public override int Read(byte[] buffer, int offset, int count)
    {
        return responseStream.Read(buffer, offset, count);
    }

    public override long Seek(long offset, SeekOrigin origin)
    {
        return responseStream.Seek(offset, origin);
    }

    public override void SetLength(long length)
    {
        responseStream.SetLength(length);
    }

    #endregion Stream

    public override void Write(byte[] buffer, int offset, int count)
    {
        char symbol;
        bool betweenTags = true;

        for (long i = 0; i < buffer.Length; i++)
        {
            symbol = Convert.ToChar(buffer[i]);

            if (symbol == '>')
            {
                betweenTags = true;
                responseStream.WriteByte(buffer[i]);
                continue;
            }

            if (betweenTags && Char.IsWhiteSpace(symbol)) continue;

            if (i > 1 && betweenTags && (Convert.ToChar(buffer[i - 1]) == ' ')) responseStream.WriteByte(buffer[i - 1]);

            responseStream.WriteByte(buffer[i]);

            betweenTags = false;
        }
    }
}
```

Module tested on servers with 2k users online, it absolutelly does not affect CPU, but also does not always give Google Page Speed Insights what it wants.

On different pages there is up to 20% savings on page size.
