---
layout: post
title: PowerShell Excell example

tags: [cmd, powershell, excel, comobject, workbooks, cells]
---

```powershell
$a = new-object -comobject excel.application

$b = $a.Workbooks.Add()
$c = $b.Worksheets.Item(1)
$c.Cells.Item(1,1) = "Hello world"
$a.Visible = $True
$b.SaveAs("C:\Users\mac\Desktop\lala.xls")
$a.Quit()
```
