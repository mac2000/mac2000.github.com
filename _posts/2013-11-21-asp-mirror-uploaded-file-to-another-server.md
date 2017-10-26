---
layout: post
title: ASP.NET Mirror Uploaded File To Another Server
tags: [iis, asp, net, mirror, storage, farm]
---

We have two sites `website1.com` and `website2.com` and want to mirror file uploaded into first site to second.

website1.com upload
-------------------

*Default.aspx*

```html
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="Default.aspx.cs" Inherits="fileupload.Default" %>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
	<title>upload</title>
</head>
<body>
	<form id="form1" runat="server">
		<asp:FileUpload runat="server" ID="fileUpload" />
		<asp:Button runat="server" ID="submit" OnClick="submit_OnClick" />
	</form>
</body>
</html>
```

*Default.aspx.cs*

```csharp
using System;
using System.IO;
using System.Net;
using System.Web.UI;

namespace fileupload
{
	public partial class Default : Page
	{
		protected void submit_OnClick(object sender, EventArgs e)
		{
			string filename = Path.Combine(Server.MapPath("~/App_Data"), fileUpload.PostedFile.FileName);

			fileUpload.PostedFile.SaveAs(filename); // save uploaded file as usual
			(new WebClient()).UploadFile(new Uri("http://mirrorer.mac.rabota.ua/Upload.ashx"), filename); // mirror uploaded file to another server
		}
	}
}
```

website2.com mirror
-------------------

*Upload.ashx.cs*

This handler used to receive upload requests from first site. There is no security or validation checks.

```csharp
using System.IO;
using System.Web;

namespace Mirrorer
{
	public class Upload : IHttpHandler
	{

		public void ProcessRequest(HttpContext context)
		{
			var file = context.Request.Files[0];
			file.SaveAs(Path.Combine(context.Server.MapPath("~/App_Data"), file.FileName));
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

Another implementation

Suppose we have farm with same two sites and separate server for static.

The idea is to save file to remote file server not to local and to not use UNC file shares.

On the static files server we have `upload.ashx` wich catches uploaded files and save them to local file system. This handler accessible only via https protocol and only with `key` request parameter.


*upload.ashx*

```csharp
using System;
using System.Web;
using System.IO;

public class upload : IHttpHandler {

	public void ProcessRequest (HttpContext context) {
		if (context.Request.HttpMethod != "POST" || context.Request.Files.Count == 0)
		{
			context.Response.StatusCode = 400;
			context.Response.End();
			return;
		}

		if (string.IsNullOrEmpty(context.Request.Params.Get("Key")) || context.Request.Params.Get("Key") != System.Configuration.ConfigurationManager.AppSettings.Get("Key"))
		{
			context.Response.StatusCode = 401;
			context.Response.End();
			return;
		}

		var file = context.Request.Files[0];

		string fileName = SanitizeFileName(Path.GetFileName(file.FileName));

		string basePath = context.Server.MapPath("~/files/");
		string baseUrl = "/files/";
		string relativePath = Path.Combine(GetSubFolderNameFromMimeType(file.ContentType).Trim('/').Trim('\\'), DateTime.Now.ToString("yyyy-MM-dd").Replace('-', '\\'));
		string dirPath = Path.Combine(basePath, relativePath);
		if (!Directory.Exists(dirPath)) Directory.CreateDirectory(dirPath);

		string filePath = Path.Combine(basePath, relativePath, fileName);

		file.SaveAs(filePath);

		context.Response.Redirect(GetFileUrl(baseUrl, relativePath, fileName));
	}

	protected string GetFileUrl(string baseUrl, string relativePath, string fileName)
	{
		return baseUrl + relativePath.Replace('\\', '/').Trim('/') + "/" + fileName;
	}

	protected string SanitizeFileName(string fileName)
	{
		return DateTime.Now.Ticks.ToString("x") + Path.GetExtension(fileName).ToLower();
		//return fileName;
	}

	protected string GetSubFolderNameFromMimeType(string mimeType)
	{
		if (mimeType.IndexOf("image") == 0) return "image";

		return "other";
	}

	public bool IsReusable {
		get {
			return false;
		}
	}

}
```

for future use here is *delete.ashx*

```csharp
using System;
using System.Web;
using System.IO;

public class delete : IHttpHandler {

	public void ProcessRequest (HttpContext context) {
		if (context.Request.HttpMethod != "POST" || string.IsNullOrEmpty(context.Request.Params["file"]))
		{
			context.Response.StatusCode = 400;
			context.Response.End();
			return;
		}

		if (string.IsNullOrEmpty(context.Request.Params.Get("Key")) || context.Request.Params.Get("Key") != System.Configuration.ConfigurationManager.AppSettings.Get("Key"))
		{
			context.Response.StatusCode = 401;
			context.Response.End();
			return;
		}

		string file = context.Server.MapPath(new Uri(context.Request.Params["file"]).PathAndQuery);

		if (File.Exists(file)) File.Delete(file);
		else
		{
			context.Response.StatusCode = 404;
			context.Response.End();
			return;
		}
		context.Response.End();
	}

	public bool IsReusable {
		get {
			return false;
		}
	}

}
```

On the main site instead of doing default save for uploaded files we will use following helper class:

*RFS.cs*

```csharp
public class RFS
{
	public string Key = string.Empty;

	public RFS(string key)
	{
		Key = key;
	}

	public Uri Upload(HttpPostedFile file) {
		NameValueCollection nvc = new NameValueCollection();
		nvc.Add("Key", Key);

		return HttpUploadFile("https://rfs.beta.rabota.ua/upload.ashx", file, nvc);
	}

	public bool Delete(string url)
	{
		return DeleteFile("https://rfs.beta.rabota.ua/delete.ashx", url);
	}

	protected Uri HttpUploadFile(string url, HttpPostedFile file, NameValueCollection additionalRequestParameters = null, string postedFileParameterName = "file")
	{
		if (additionalRequestParameters == null)
		{
			additionalRequestParameters = new NameValueCollection();
		}

		string boundary = "---------------------------" + DateTime.Now.Ticks.ToString("x");
		byte[] boundarybytes = System.Text.Encoding.ASCII.GetBytes("\r\n--" + boundary + "\r\n");

		HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
		request.ContentType = "multipart/form-data; boundary=" + boundary;
		request.Method = "POST";
		request.KeepAlive = true;
		request.Credentials = System.Net.CredentialCache.DefaultCredentials;

		Stream requestStream = request.GetRequestStream();

		#region Write Additional Request Parameters
		string formdataTemplate = "Content-Disposition: form-data; name=\"{0}\"\r\n\r\n{1}";
		foreach (string key in additionalRequestParameters.Keys)
		{
			requestStream.Write(boundarybytes, 0, boundarybytes.Length);
			string formitem = string.Format(formdataTemplate, key, additionalRequestParameters[key]);
			byte[] formitembytes = System.Text.Encoding.UTF8.GetBytes(formitem);
			requestStream.Write(formitembytes, 0, formitembytes.Length);
		}
		#endregion

		#region Write File
		requestStream.Write(boundarybytes, 0, boundarybytes.Length);
		string headerTemplate = "Content-Disposition: form-data; name=\"{0}\"; filename=\"{1}\"\r\nContent-Type: {2}\r\n\r\n";
		string header = string.Format(headerTemplate, postedFileParameterName, file.FileName, file.ContentType);
		byte[] headerbytes = System.Text.Encoding.UTF8.GetBytes(header);
		requestStream.Write(headerbytes, 0, headerbytes.Length);

		byte[] buffer = new byte[4096];
		int bytesRead = 0;
		while ((bytesRead = file.InputStream.Read(buffer, 0, buffer.Length)) != 0)
		{
			requestStream.Write(buffer, 0, bytesRead);
		}
		file.InputStream.Close();
		#endregion

		#region Close stream
		byte[] trailer = System.Text.Encoding.ASCII.GetBytes("\r\n--" + boundary + "--\r\n");
		requestStream.Write(trailer, 0, trailer.Length);
		requestStream.Close();
		#endregion

		HttpWebResponse response = (HttpWebResponse)request.GetResponse();
		if (response.StatusCode != HttpStatusCode.OK)
		{
			throw new HttpException((int)response.StatusCode, response.StatusDescription);
		}

		Uri uri = response.ResponseUri;

		if (response != null)
		{
			response.Close();
		}

		requestStream = null;
		request = null;
		response = null;

		return uri;
	}

	protected bool DeleteFile(string url, string fileUrl)
	{
		using (var wb = new WebClient())
		{
			var data = new NameValueCollection();
			data["file"] = fileUrl;
			data["Key"] = Key;

			var response = wb.UploadValues(url, "POST", data);
		}

		return true;
	}
}
```

So now our handle method will be something like this:

```csharp
protected void submitButton_Click(object sender, EventArgs e)
{
	if (fileUpload.HasFile)
	{
		try
		{
			//TODO: validate file here
			//Default way to save uploaded file: fileUpload.SaveAs(Path.Combine(Server.MapPath("~/"), Path.GetFileName(fileUpload.FileName)));
			public RFS rfs = new RFS("HelloWorld");
			Uri uri = rfs.Upload(fileUpload.PostedFile);

			img.ImageUrl = uri.ToString();
			img.Visible = true;
		}
		catch (Exception ex)
		{
			literal.Text = "ERROR: " + ex.Message;
		}
	}
}
```
