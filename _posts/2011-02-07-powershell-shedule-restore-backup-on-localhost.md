---
layout: post
title: PowerShell shedule restore backup on localhost

tags: [admin, administration, auto, automatic, backup, batch, cmd, powershell, ps, ps1, restore, shell]
---

So I want get fresh images etc from server:

```powershell
Set-Variable -name SBS_ARCH_PATH -value "\\ProductServer\wwwroot" -option constant
Set-Variable -name SMTP_SERVER -value "192.168.0.1" -option constant
Set-Variable -name EMAIL_FROM -value "alexandrm@rabota.ua" -option constant
Set-Variable -name EMAIL_TO -value "alexandrm@rabota.ua" -option constant
Set-Variable -name SEVEN_ZIP_EXE_PATH -value "C:\Program Files\7-Zip\7z.exe" -option constant

# check is SBS server is accessible
function isSbsAccessible()
{
    return (Test-Path $SBS_ARCH_PATH)
}

# retrive last archive file name from SBS
function getLastArchiveName()
{
    $path = $SBS_ARCH_PATH + "\*.rar"
    $path = ls $path | sort-object Name | select-object -last 1 | foreach-object {$_.Name}
    return $path
}

# get last archive path on SBS
function getLastArchivePath($path)
{
    return ($SBS_ARCH_PATH + "\" + $path)
}

# get target archive path
function getTargetArchPath()
{
    $path = $env:Temp
    $path = $path + "\rabotauabackup.rar"
    return $path
}

# send email
function sendEmail($body)
{
    $subject = ("["+(Get-Date).ToString()+"] files update job report")
    $smtp = new-object Net.Mail.SmtpClient($SMTP_SERVER)
    $mail = new-object Net.Mail.MailMessage($EMAIL_FROM, $EMAIL_TO, $subject, $body)
    $mail.IsBodyHtml = $True
    $smtp.Send($mail)
}

if(isSbsAccessible) { # if SBS is accessible
    Write-Host "All ok, SBS is accessebile" -foregroundcolor green

    # generate pathes
    $lastArchName = getLastArchiveName
    $lastArchPath = getLastArchivePath $lastArchName
    $targetPath = getTargetArchPath

    Write-Host "Copying backup" -foregroundcolor yellow
    copy $lastArchPath $targetPath

    if(Test-Path $targetPath) { # check is file was copied

        Write-Host "Extracting files" -foregroundcolor yellow

        if(Test-Path "C:\Program Files\7-Zip\") { # is there 7z?
            set-alias sz "C:\Program Files\7-Zip\7z.exe"
            #$extractPath = $env:Temp
            #$extractPath = $extractPath + "\rabotauabackup"
            #sz x -pPASSWORD $targetPath -o$extractPath -r -aou
            sz x -pPASSWORD $targetPath -oC:\inetpub\wwwroot\rabotauabackup -r -aou

            robocopy C:\inetpub\wwwroot\rabotauabackup\Inetpub\wwwroot\BlogEngine\App_Data\files C:\Users\mac\Rabota.UA\trunk\Version\Rabota2.CMS\BlogEngine.Web\App_Data\files /MIR

            Write-Host "SUCCESS" -foregroundcolor green

            $body = "<h3 style='color:green'>SUCCESS</h3>"
            $body += "<p>Files updated from <a href='" + $lastArchPath + "'>" + $lastArchName + "</a></p>"
            sendEmail $body
        }
        else { # if therre is no 7z
            $body = "<h3 style='color:red'>FAILURE</h3>"
            $body += "<p>7Zip was not found</p>"
            sendEmail $body
        }

    } else { # file was not copied
        $body = "<h3 style='color:red'>FAILURE</h3>"
        $body += "<p>Backup file was not copied from<a href='" + $lastArchPath + "'>" + $lastArchName + "</a></p>"
        sendEmail $body
    }

} else { # if SBS is NOT accessible
    Write-Host "SBS is not accessibile" -foregroundcolor red
    $body = "<h3 style='color:red'>FAILURE</h3>"
    $body += "<p>SBS was not accessible</p>"
    sendEmail $body
}
```

Then all u need to do is register this script in sheduler
