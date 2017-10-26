---
layout: post
title: Check HTML markup in ResX files
tags: [asp, resx, validate, i18n, l10n, localization, internationalization]
---

So, we got almost one thousand of resource files translated for public part of our project.

Unfortunatelly on such huge amounts of data there is big chanse for human mistakes. We broke few important pages in our site with not valid html markup from resx files.

So here is nice and simple way to check if there any errors in resource files html:

```powershell
[System.Reflection.Assembly]::LoadFrom((Join-Path $PSScriptRoot -ChildPath 'HtmlAgilityPack.dll')) | Out-Null
$doc = New-Object HtmlAgilityPack.HtmlDocument


$files = Get-ChildItem -Path C:\Rabota.UA\trunk\Version\Rabota2.WebUI -File -Include *.resx -Recurse -ErrorAction SilentlyContinue

$errors = @()
foreach($file in $files) {
    Write-Verbose $file.FullName

    $items = ([xml]$xml = Get-Content $file.FullName -Encoding UTF8).root.data

    foreach($item in $items) {
        $doc.LoadHtml($item.value)
        if($doc.ParseErrors.Count -gt 0) {
            Write-Host $file.FullName -ForegroundColor Yellow -NoNewline
            Write-Host (' ' + $item.name) -ForegroundColor Cyan

            $doc.ParseErrors | ft -AutoSize

            $errors += $doc.ParseErrors
        }
    }

    Write-Progress -Activity 'Checking HTML' -Status $file.FullName -PercentComplete ( [Array]::IndexOf($files, $file) / $files.Count * 100 )
}

if($errors.Count -gt 0) {
    Write-Host ('Found ' + $errors.Count + ' errors') -ForegroundColor Red
} else {
    Write-Host 'All seems to be OK' -ForegroundColor Green
}
```

and its output:

```
C:\Rabota.UA\trunk\Version\Rabota2.WebUI\App_GlobalResources\cvbuilder.en.resx FinanceSkillsRightExample

        Code Line LinePosition Reason                      SourceText StreamPosition
        ---- ---- ------------ ------                      ---------- --------------
TagNotClosed    3            1 End tag </ul> was not found                        34
TagNotClosed    7           57 End tag </ul> was not found                       266


C:\Rabota.UA\trunk\Version\Rabota2.WebUI\App_GlobalResources\cvbuilder.en.resx ITSkillsRightExample

        Code Line LinePosition Reason                      SourceText StreamPosition
        ---- ---- ------------ ------                      ---------- --------------
TagNotClosed    3            1 End tag </ul> was not found                        34
TagNotClosed    7           57 End tag </ul> was not found                       371

...

C:\Rabota.UA\trunk\Version\Rabota2.WebUI\Controls\CvBuilder\App_LocalResources\StepThree.ascx.resx Incomplete2

                Code Line LinePosition Reason                      SourceText StreamPosition
                ---- ---- ------------ ------                      ---------- --------------
EndTagNotRequired    1           41 End tag </> is not required <                      40


C:\Rabota.UA\trunk\Version\Rabota2.WebUI\Controls\CvBuilder\App_LocalResources\StepThree.ascx.uk.resx Incomplete2

                Code Line LinePosition Reason                      SourceText StreamPosition
                ---- ---- ------------ ------                      ---------- --------------
EndTagNotRequired    1           37 End tag </> is not required <                      36


Found 52 errors
```

The reason why I so excited about this stuf - think how could such script to be used to detect broken html in asp webforms user controls!

Note: for this to work you will need [HtmlAgilityPack.dll](http://htmlagilitypack.codeplex.com/), also do not forget to change path to root folder of your porject



Check broken HTML in WebForms User Controls
-------------------------------------------

```powershell
[System.Reflection.Assembly]::LoadFrom((Join-Path $PSScriptRoot -ChildPath 'HtmlAgilityPack.dll')) | Out-Null
$doc = New-Object HtmlAgilityPack.HtmlDocument


$files = Get-ChildItem -Path C:\Rabota.UA\trunk\Version\Rabota2.WebUI -File -Include *.ascx, *.aspx, *.master -Recurse -ErrorAction SilentlyContinue

$errors = @()
foreach($file in $files) {
    Write-Verbose $file.FullName

    $doc.LoadHtml((Get-Content $file.FullName -Encoding UTF8))

    if($doc.ParseErrors.Count -gt 0) {
        Write-Host $file.FullName -ForegroundColor Yellow

        $doc.ParseErrors | ft -AutoSize

        $errors += $doc.ParseErrors
    }

    Write-Progress -Activity 'Checking HTML' -Status $file.FullName -PercentComplete ( [Array]::IndexOf($files, $file) / $files.Count * 100 )
}

if($errors.Count -gt 0) {
    Write-Host ('Found ' + $errors.Count + ' errors') -ForegroundColor Red
} else {
    Write-Host 'All seems to be OK' -ForegroundColor Green
}
```

What you should be avare of - it is not silver bullet and can not find matchin tags for stuffs like Repeater > Header(Footer)Template
