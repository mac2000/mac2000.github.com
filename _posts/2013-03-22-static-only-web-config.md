---
layout: post
title: Static only web.config
tags: [asp, static, web.config]
---

IIS 6.0 can not map virtual directories via UNC share with specific credentials until you will not convert it into application.

So if all you need is to serve static files only, here is simple example:

```xml
<?xml version="1.0" encoding="utf-8"?>

<configuration>
	<system.web>
		<compilation>
			<assemblies>
				<!-- remove any parent applications assemblies -->
				<clear/>
			</assemblies>
		</compilation>
		<httpModules>
			<!-- remove all modules -->
			<clear/>
		</httpModules>
		<httpHandlers>
			<!-- remove all handlers -->
			<clear/>
			<!-- add default handler to precess requests -->
			<add path="*" verb="GET,HEAD,POST" type="System.Web.DefaultHttpHandler" validate="True" />
		</httpHandlers>
	</system.web>
</configuration>
```

Default handler and all other defaults can be found here:

```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\web.config
```