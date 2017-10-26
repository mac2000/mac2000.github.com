---
layout: post
title: C# XmlSerializer XSL

tags: [.net, c#, serialize, StreamWriter, xml, xmlserializer, XmlTextWriter, xsl]
---

```csharp
XmlSerializer serializer = new XmlSerializer(typeof(List<ReportItem>));

using (StreamWriter streamWriter = new StreamWriter("reportItems.xml"))
{
    using (XmlTextWriter writer = new XmlTextWriter(streamWriter))
    {
        writer.WriteProcessingInstruction("xml", "version=\"1.0\" encoding=\"utf-8\"");
        writer.WriteProcessingInstruction("xml-stylesheet", "type=\"text/xsl\" href=\"reportItems.xsl\"");
        serializer.Serialize(writer, reportItems);
    }
}
/*TextWriter textWriter = new StreamWriter("reportItems.xml");
serializer.Serialize(textWriter, reportItems);
textWriter.Close();*/
```
