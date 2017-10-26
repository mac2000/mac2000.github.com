---
layout: post
title: ASHX to create graphs via graphviz
tags: [ashx, asp.net, asp, c#, png, graphviz, dot]
---

http://www.graphviz.org/ - tools to draw graphs from text.

Here is sample file `test.dot`:

```
digraph G {
	"A"->"B"
	"C"->"A"
	"D"->"C"
	"E"->"C"
	"F"->"D"
}
```

Now you can run:

```sh
dot test.dot -Tpng > test.png
```

To generate png image from it.

Here is sample code to do it in ashx:

```csharp
<%@ WebHandler Language="C#" Class="dot" %>

using System;
using System.Web;
using System.Data;
using System.Data.SqlClient;
using System.Collections.Generic;
using System.IO;
using System.Diagnostics;
using System.Runtime.Serialization.Diagnostics;
using System.Drawing;
using System.Drawing.Imaging;

public class dot : IHttpHandler
{
	public void ProcessRequest(HttpContext context)
	{
		context.Response.ContentType = "image/png";

		try
		{
			if (String.IsNullOrEmpty(context.Request.Params["ID"])) throw new Exception("ID is required");

			context.Response.BinaryWrite(Dot2Png(DataSet2Dot(GetData(int.Parse(context.Request.Params["ID"])))));
		}
		catch (Exception ex)
		{
			context.Response.BinaryWrite(GenerateMessage(ex.Message));
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

	protected byte[] Dot2Png(string dot)
	{
		string file = Path.GetTempFileName();
		string png = Path.GetTempFileName();
		string command = @"""C:\Program Files (x86)\Graphviz2.36\bin\dot.exe"" " + file + " -Tpng > " + png;

		File.WriteAllText(file, dot);

		System.Diagnostics.ProcessStartInfo procStartInfo = new System.Diagnostics.ProcessStartInfo("cmd", "/c " + command);
		procStartInfo.RedirectStandardOutput = true;
		procStartInfo.UseShellExecute = false;
		procStartInfo.CreateNoWindow = true;
		System.Diagnostics.Process proc = new System.Diagnostics.Process();
		proc.StartInfo = procStartInfo;
		proc.Start();
		string result = proc.StandardOutput.ReadToEnd();

		byte[] bytes = File.ReadAllBytes(png);

		File.Delete(png);
		File.Delete(file);

		return bytes;
	}

	protected string DataSet2Dot(DataSet ds)
	{
		List<string> items = new List<string>();

		foreach (DataRow edge in ds.Tables[1].Rows)
		{
			string ParentName = "";
			string ChildName = "";

			foreach (DataRow node in ds.Tables[0].Rows)
			{
				if (node["ID"].ToString() == edge["ParentID"].ToString())
				{
					ParentName = node["Name"].ToString();
				}
				else if (node["ID"].ToString() == edge["ChildID"].ToString())
				{
					ChildName = node["Name"].ToString();
				}
			}

			items.Add(string.Format("\t\"{0}\"->\"{1}\"", ParentName, ChildName));
		}

		return string.Format($"digraph G { { {0} {1} {0} } } {0}", Environment.NewLine, string.Join(Environment.NewLine, items.ToArray()));
	}

	/// <summary>
	/// Retrieve dataset with two tables.
	///
	/// Table[0] (ID, Name)
	/// Table[1] (ParentID, ChildID)
	/// </summary>
	/// <param name="ID"></param>
	/// <returns></returns>
	protected DataSet GetData(int ID)
	{
		DataSet ds = new DataSet();
		using (var conn = new SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings["RabotaUA"].ToString()))
		using (
			var command = new SqlCommand("spTagMaster_Structure", conn)
			{
				CommandType = CommandType.StoredProcedure
			})
		{
			conn.Open();
			command.Parameters.Add(new SqlParameter("@ID", ID));
			SqlDataAdapter da = new SqlDataAdapter();
			da.SelectCommand = command;
			da.Fill(ds);
			conn.Close();
		}
		return ds;
	}

	public bool IsReusable
	{
		get
		{
			return false;
		}
	}
}
```

This code expects to retrieve two tables from `spTagMaster_Structure` stored procedure, one of which should have ID, Name columns and second - ParentID, ChildID.

