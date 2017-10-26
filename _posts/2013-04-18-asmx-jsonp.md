---
layout: post
title: Call asmx from another domain using jsonp
tags: [asmx, webservice, jsonp]
---

Step 1. Allow GET requests for Web Services

**Web.config**

```xml
<?xml version="1.0"?>
<configuration>
	<system.web>
		<compilation debug="true" targetFramework="4.0"/>
		<webServices>
			<protocols>
				<!-- Allow GET request for Web Services -->
				<add name="HttpGet" />
			</protocols>
		</webServices>
	</system.web>
</configuration>
```

Step 2. Additional service methods

**Calculator.cs**

```csharp
using System;
using System.Collections.Generic;
using System.Web;
using System.Web.Script.Services;
using System.Web.Services;

[WebService(Namespace = "http://tempuri.org/")]
[WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
[ScriptService]
public class Calculator : System.Web.Services.WebService {

	// Normal behaviour
	[WebMethod]
	public int Add(int a, int b) {
		return a + b;
	}

	// JSONP
	// Do not forget to add "<add name="HttpGet" />" to Web.config\configuration\webServices\protocols
	// Notice that method returns nothing (void)
	[WebMethod]
	public void JSAdd(int a, int b)
	{
		int original_result = Add(a, b);

		// Prepare
		string callback = HttpContext.Current.Request.Params["callback"];
		string json = "{c: " + original_result + "}";//Newtonsoft.Json.JsonConvert.SerializeObject(...);
		string response = string.IsNullOrEmpty(callback) ? json : string.Format("{0}({1});", callback, json);

		// Response
		HttpContext.Current.Response.ContentType = "application/json";
		HttpContext.Current.Response.Write(response);
	}
}
```

Now you can query your service like this:

```js
jQuery.getJSON('http://localhost:54161/Calculator.asmx/JSAdd?a=2&b=2&callback=?', function(data){
	alert(data.c); // 4
});
```

Do not forget `callback` argument when requesting web service from another domain.