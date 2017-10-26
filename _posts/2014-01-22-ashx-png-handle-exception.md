---
layout: post
title: PNG ASHX handle Exceptions
tags: [ashx, asp.net, asp, c#, png, bitmap, exception, binarywrite, font, fontstyle, size, fromimage, drawing2d, smoothingmode, antialias, solidbrush, memorystream, imageformat]
---

Suppose that we have some ashx that responses with png images.

Here is way to handle exceptions:

```csharp
<%@ WebHandler Language="C#" Class="example" %>

using System;
using System.Web;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;

public class example : IHttpHandler {

	public void ProcessRequest (HttpContext context) {
		context.Response.ContentType = "image/png";

		try
		{
			if (String.IsNullOrEmpty(context.Request.Params["ID"])) throw new Exception("ID is required");

			// your image generation code here...
		}
		catch (Exception ex)
		{
			context.Response.BinaryWrite(GenerateMessage(ex.Message));
		}
	}

	public bool IsReusable {
		get {
			return false;
		}
	}

	protected byte[] GenerateMessage(string message)
	{
		Bitmap objBmpImage = new Bitmap(1, 1);
		int intWidth = 0;
		int intHeight = 0;
		Font objFont = new Font("Arial", 13, System.Drawing.FontStyle.Bold, System.Drawing.GraphicsUnit.Pixel);
		Graphics objGraphics = Graphics.FromImage(objBmpImage);
		intWidth = (int)objGraphics.MeasureString(message, objFont).Width;
		intHeight = (int)objGraphics.MeasureString(message, objFont).Height;
		objBmpImage = new Bitmap(objBmpImage, new Size(intWidth, intHeight));
		objGraphics = Graphics.FromImage(objBmpImage);
		objGraphics.Clear(Color.White);
		objGraphics.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;
		objGraphics.TextRenderingHint = System.Drawing.Text.TextRenderingHint.AntiAlias;
		objGraphics.DrawString(message, objFont, new SolidBrush(Color.FromArgb(102, 102, 102)), 0, 0);
		objGraphics.Flush();

		MemoryStream mem = new MemoryStream();
		objBmpImage.Save(mem, ImageFormat.Png);

		return mem.ToArray();
	}

}
```
