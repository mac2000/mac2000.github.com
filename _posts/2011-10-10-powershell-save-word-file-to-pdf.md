---
layout: post
title: PowerShell save word file to pdf

tags: [convert, converter, doc, docx, ghostscript, gs, gswin32c, pdf, powershell, ps, ps1, word, word2pdf]
---

```powershell
$word = new-object -ComObject "word.application"

$doc = $word.documents.open("C:\Users\AlexandrM\Desktop\TempCVArch\CV Кулакова C&B.doc")
$doc.SaveAs([ref] "C:\Users\AlexandrM\Desktop\1.pdf", [ref] 17)
$doc.Close()
$word.Quit()
```

Такая штука нужна чтобы потом через ghostscript сделать вот так:

```sh
gswin32c -q -dQUIET -dPARANOIDSAFER -dBATCH -dNOPAUSE -dNOPROMPT -sDEVICE=pngalpha -sOutputFile=1-%d.png 1.pdf
```

что нагенерит рисунки с страницами из pdf, в общем можно сделать свой google docs viewer

**pdf2png.ps1:**

```powershell
# preferred dimensions
# only width is required, height is calculated via A4 paper dimensions with aspect ratio
$width = 658
$height = [math]::Round((297 / 210) * $width)

# first argument must be existing file path
if(($args.count -gt 0) -and (Test-Path $args[0])) {

    # define some vars
    $input_file = $args[0];                                                                # will be: "C:\Users\AlexandrM\Desktop\test\cv1.docx"
    $random_name = "_tmp_"+[system.io.Path]::GetRandomFileName()
    $input_file_name = [system.io.Path]::GetFileNameWithoutExtension($input_file)          # will be: "cv1"
    $input_file_extension = [system.io.Path]::GetExtension($input_file)                    # will be: ".docx"
    $input_file_dir = [system.io.Path]::GetDirectoryName($input_file) + "\"                # will be: "C:\Users\AlexandrM\Desktop\test\"
    $pdf_file = $input_file_dir + $random_name + ".pdf"                                    # will be: "C:\Users\AlexandrM\Desktop\test\cv1.pdf"
    $png_file = $input_file_dir + $input_file_name + ".png"                                # will be: "C:\Users\AlexandrM\Desktop\test\cv1.png"
    $png_parts_pattern_for_ghostscript = $input_file_dir + $random_name + "---gs-%03d.png" # will be: "C:\Users\AlexandrM\Desktop\test\cv1---gs-001.png"
    $png_parts_pattern_for_imagemagick = $input_file_dir + $random_name + "---gs-*.png"

    # if not pdf - convert via word
    if(!($input_file_extension -eq ".pdf")) {
        $word = new-object -ComObject "word.application"
        $doc = $word.documents.open($input_file)
        $doc.SaveAs([ref] $pdf_file, [ref] 17)
        $doc.Close()
        $word.Quit()
    }

    # if pdf created
    if(Test-Path $pdf_file) {
        # extract pages from it as png images
        invoke-expression -Command "gswin32c.exe -dNOPAUSE -q -dBATCH -sDEVICE=png16m -dPDFFitPage -dFIXEDMEDIA -dDEVICEWIDTHPOINTS=$width -dDEVICEHEIGHTPOINTS=$height -sOutputFile=$png_parts_pattern_for_ghostscript $pdf_file"
        # merge all pages into one image
        $montage_command = "montage -tile 1 -geometry " + $width + " -interlace PNG " + $png_parts_pattern_for_imagemagick + " `"" + $png_file + "`""
        invoke-expression -Command $montage_command
        # remove unneeded files
        Remove-Item $png_parts_pattern_for_imagemagick
        Remove-Item $pdf_file
    }
}
```

На выходе этого срипта - рисунок который уже можно подключать на страничку - собственно вот и свой google docs viewer получился

**How to get PID of Word.application**

```powershell
Add-Type -TypeDefinition @"
using System;
using System.Runtime.InteropServices;

public static class Win32Api
{
[System.Runtime.InteropServices.DllImportAttribute( "User32.dll", EntryPoint =  "GetWindowThreadProcessId" )]
public static extern int GetWindowThreadProcessId ( [System.Runtime.InteropServices.InAttribute()] System.IntPtr hWnd, out int lpdwProcessId );

[DllImport("User32.dll", CharSet = CharSet.Auto)]
public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
}
"@

$word = New-Object -ComObject "Word.application"

$caption = [guid]::NewGuid()
$word.Caption = $caption
$HWND = [Win32Api]::FindWindow("OpusApp", $caption)
$wordPid = [IntPtr]::Zero
[Win32Api]::GetWindowThreadProcessId($HWND, [ref]$wordPid);

$word.visible = $true
Start-Sleep -Seconds 5

Stop-Process -Id $wordPid -ErrorAction SilentlyContinue
```

