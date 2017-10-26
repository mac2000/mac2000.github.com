---
layout: post
title: Apache Tika converts doc to text
tags: [apache, tike, converter, convert, doc, docx, txt, tool, lib, jar, java]
---

http://tika.apache.org/ - here is tool site, just download jar file somewhere and ensure that java is installed.

Now you can convert documents, like this:

```sh
java -jar tika-app-1.4.jar test.doc > test.html
java -jar tika-app-1.4.jar --text test.doc > test.txt
```

This tool can convert following formats: html, doc, docx, xml, odf, pdf, epub and others.
