---
layout: post
title: ASP.Net Clear OutputCache

tags: [.net, asp.net, cache, iis, performance, vb]
---

**OutputCached.aspx**

```csharp
<%@ Page Language="VB"  Debug="true"  ContentType="text/html" ResponseEncoding="iso-8859-1" %>
<%@ OutputCache Duration="3600" Location="Server"  VaryByParam="none" %>
<html>
    <head>
        <title>This page is from OutputCache</title>
    </head>
    <body>
        <h1><%=Now()%></h1>
    </body>
</html>
```

**ClearOutputCache.aspx**

```csharp
<%@ Page Language="VB"  Debug="true"  ContentType="text/html" ResponseEncoding="iso-8859-1" %>
<script runat="server">
    Private Sub Button1_Click(ByVal sender As Object, ByVal e As System.EventArgs)
        Dim path As String
        path="/AbosoluteVitualPath/OutputCached.aspx"
        HttpResponse.RemoveOutputCacheItem(path)
    End Sub
</script>
<html>
    <head>
    </head>
    <body>
        <p>Clear Output Cache</p>
        <form id="Form1" method="post" runat="server">
            <asp:Button id="Button1" OnClick="Button1_Click" runat="server" Text="Button"></asp:Button>
        </form>
    </body>
</html>
```
