---
layout: post
title: Use Powershell to detect RESX duplicate keys
tags: [powershell, resx, duplicate]
---

What is wrong with localization in ASP.NET? - Everything.

If you project stop compiling and you get some strange errors like "Unknown server control" there is probably errors in your resx files.

Visual Studio from time to time can show you duplicate keys but not always, especialy if they are in different cases. Also while editing resx in visual editor visual studio not showing duplicates at all.

To prevent this from happening you should check your resx files for duplicate keys almost manualy, so here is little nice way to automate this:

```powershell
$items = @()
Get-ChildItem -Path C:\Rabota.UA\trunk\Version\Rabota2.WebUI -File -Include *.resx -Recurse -ErrorAction SilentlyContinue | %{
    $duplicates = ([xml]$xml = Get-Content $_.FullName).root.data | select @{n='NameLowerCased';e={ $_.Name.ToLower() }} | group-object NameLowerCased | select Name, Count | Where-Object Count -gt 1

    if($duplicates.Count -gt 0) {

        $item = New-Object psobject

        $item | Add-Member NoteProperty FullName $_.FullName
        $item | Add-Member NoteProperty Count $duplicates.Count
        $item | Add-Member NoteProperty Duplicates ([String]::Join(', ', ($duplicates | select -ExpandProperty Name)))

        $items += $item
    }
}
$items | ft -AutoSize
```

Which will output womething like:

```
FullName                                                                                          Count Duplicates
--------                                                                                          ----- ----------
C:\Rabota.UA\trunk\Version\Rabota2.WebUI\Controls\Vacancy\App_LocalResources\AfterApply.ascx.resx     2 tip5.text, ltback
```

Again and again thank you MS for giving us such powerfull tool like Powershell :)
