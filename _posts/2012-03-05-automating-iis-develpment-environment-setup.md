---
layout: post
title: Automating IIS develpment environment setup

tags: [AddAccessRule, admin, administration, application, asp.net, automation, automatization, COMPUTERNAME, Copyhere, env, Get-Acl, Get-ChildItem, Get-Content, Get-WebSite, iis, Import-Module, Invoke-Expression, IsInRole, Join-Path, Measure-Object, New-Item, New-WebApplication, New-WebBinding, New-WebSite, New-WebVirtualDirectory, powershell, PromptForChoice, promt, ps1, Set-Acl, shell, Split-Path, Stop-Website, Test-Path, unzip, USERDOMAIN, w3svc, WebAdministration, Write-Host, xml, zip]
---

We're working on few ASP.NET projects, so have need in some automatization.

Here is some brief notes and links:

**Run powershell script by pass execution policy from cmd file**

```sh
isadmin.cmd
PowerShell.exe -executionpolicy bypass -File c:\users\alexandrm\desktop\isadmin.ps1
```

**Make symlink to folder (on left - link, on right - directory)**

```sh
mklink /D C:\inetpub\wwwroot\CvPhotos D:\CvPhotos
```

**To setup all needed components run this command:**

```sh
start /w pkgmgr /iu:IIS-WebServerRole;IIS-WebServer;IIS-ApplicationDevelopment;IIS-CommonHttpFeatures;IIS-HealthAndDiagnostics;IIS-Performance;IIS-Security;IIS-WebServerManagementTools;IIS-RequestFiltering;IIS-HttpCompressionStatic;IIS-HttpLogging;IIS-RequestMonitor;IIS-LoggingLibraries;IIS-HttpTracing;IIS-DefaultDocument;IIS-DirectoryBrowsing;IIS-HttpErrors;IIS-HttpRedirect;IIS-StaticContent;IIS-NetFxExtensibility;IIS-ASPNET;IIS-ISAPIExtensions;IIS-ISAPIFilter;IIS-ManagementConsole;IIS-ManagementScriptingTools
```

Links:

http://samag.ru/archive/article/1072

http://technet.microsoft.com/en-us/library/ff716257%28v=ws.10%29.aspx

**Setup all via Web Platform Installer**

```powershell
$source = "http://www.iis.net/community/files/webpi/webpicmd_x86.zip"
$destination = "C:\Users\AlexandrM\Desktop\webpicmd_x86.zip"
$wc = New-Object System.Net.WebClient
$wc.DownloadFile($source, $destination)
$shell_app=New-Object -com shell.application
$filename = "C:\Users\AlexandrM\Desktop\webpicmd_x86.zip"
$zip_file = $shell_app.namespace($filename)
New-Item "C:\Users\AlexandrM\Desktop\webpicmd" -type directory
$destination = $shell_app.namespace("C:\Users\AlexandrM\Desktop\webpicmd")
$destination.Copyhere($zip_file.items())

C:\Users\user\Desktop\webpicmd\WebpiCmdLine.exe /Products: NETFramework4, KB980423-Win7, KB983484-Win7, ASPNET, DefaultDocument, DirectoryBrowse, HTTPErrors, HTTPLogging, IIS7, IISManagementConsole, ISAPIExtensions, ISAPIFilters, LoggingTools, MVC, MVC2, MVC3, MVC3Loc, MVC3Runtime, NETExtensibility, NETFramework20SP2, NETFramework35, Plan9, Plan9Loc, PowerShell, PowerShell2, RequestFiltering, RequestMonitor, StaticContent, StaticContentCompression, IISManagementScriptsAndTools /SuppressReboot /AcceptEula
```

Links:

http://msdn.microsoft.com/en-us/library/windowsazure/gg433092.aspx

http://blogs.iis.net/satishl/archive/2011/01/26/webpi-command-line.aspx

### Powershell tips

**Check is admin**

```powershell
$principal = new-object System.Security.Principal.WindowsPrincipal([System.Security.Principal.WindowsIdentity]::GetCurrent())
$isAdmin = $principal.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
if(-not $isAdmin) {
    Write-Host "You are NOT admin" -ForegroundColor Red
} else {
    Write-Host "You are admin" -ForegroundColor Green
}
```

**Retrieve environment variables**

```powershell
Get-ChildItem Env:
Get-ChildItem Env:SystemRoot | Select -expand Value
```

_Some variables_

```
COMPUTERNAME                   MAC-PC
SystemDrive                    C:
SystemRoot                     C:\Windows
TEMP                           C:\Users\ALEXAN~1\AppData\Local\Temp
TMP                            C:\Users\ALEXAN~1\AppData\Local\Temp
USERDNSDOMAIN                  RABOTA.LOCAL
USERDOMAIN                     RABOTA
USERNAME                       AlexandrM
USERPROFILE                    C:\Users\AlexandrM
```

**Include powershell file**

```sh
    . C:\Users\user\Desktop\hosts.ps1
```

_Helper_

```powershell
$currentDirectory = (Split-Path -Path $MyInvocation.MyCommand.Definition -Parent) + "\"
. ($currentDirectory + "inc.ps1")
```

**Check that .NET 4.0 installed**

```powershell
Test-Path "${Env:ProgramFiles(x86)}\Microsoft ASP.NET\ASP.NET MVC 3"
```

**Press any key to continue**

```powershell
Write-Host "Press any key to continue ..."
$x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
```

**Login info**

```powershell
if((Get-ChildItem Env:USERDOMAIN | Select -Expand Value) -ne (Get-ChildItem Env:COMPUTERNAME | Select -Expand Value)) {
    $logonServer = Get-ChildItem Env:USERDOMAIN | Select -Expand Value
} else {
    $logonServer = Get-ChildItem Env:COMPUTERNAME | Select -Expand Value
}
$userName = Get-ChildItem Env:USERNAME | Select -Expand Value
```


### Scheduled tasks

http://poshtips.com/2011/04/07/use-powershell-to-create-a-scheduled-task/

http://technet.microsoft.com/en-us/library/cc725744(v=ws.10).aspx

**Creating tasks examples**

```sh
schtasks.exe /Create /TN UpdateLocalAssets /RU user-PC\user /SC DAILY /ST 09:00 /TR "C:\Users\user\Desktop\schedule.cmd"
schtasks.exe /Create /TN UpdateLocalAssets /RU system /SC DAILY /ST 09:00 /TR "C:\Users\user\Desktop\schedule.cmd"
```

Automating
----------

Main idea is to have some scripts that will install all needed sites on local
machine and make changes to hosts file.

We have config.xml file, like this one:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <users>
        <user name="sv" ip="192.168.5.51" />
        <user name="mac" ip="192.168.5.52" />
        <user name="beta" ip="192.168.5.53" />
    </users>
    <sites>
        <site domain="rabota.ua" />

        <!-- Admin sites -->
        <site domain="admin.rabota.ua" />

        <!-- Micro sites -->
        <site domain="job-in-kiev.com.ua" />
        <site domain="job-in-dp.com.ua" />
        <site domain="job-in-kharkov.com.ua" />
        <site domain="job-in-lviv.com.ua" />
        <site domain="job-in-odessa.com.ua" />
        <site domain="job-in-donetsk.com.ua" />

        <site domain="gojobs.ru" />

        <!-- Channels -->
        <site domain="finance.rabota.ua" />
        <site domain="pharma.rabota.ua" />
        <site domain="student.rabota.ua" />
        <site domain="marketing.rabota.ua" />
        <site domain="it.rabota.ua" />
        <site domain="top.rabota.ua" />

        <!-- Channels for partners -->
        <site domain="rabota.investgazeta.net" />
        <site domain="rabota.itc.ua" />
        <site domain="rabota.delo.ua" />
        <site domain="rabota.reklamaster.com" />
        <site domain="rabota.ko.com.ua" />
    </sites>
    <projects>
        <!--
        Each project may have its own setup script in scripts/projects folder
        -->
        <project name="rabota.ua" />
        <project name="micro" />
        <project name="gojobs" />
        <project name="channels" />
        <project name="admin" />
    </projects>
</config>
```

In config, we have section with users and their ip addresses (will be used to create hosts file records).

Also there is sites section that also will be used for creating hosts files.

Projects section contains project names that will be added as sites to IIS, each project has its own setup script.

Hosts
-----

SetupHosts.cmd will setup all needed hosts on your machine, so you will be able to work with local sites and check sites on other developer machines.

**SetupHosts.cmd**

```sh
@ECHO OFF
REM http://stackoverflow.com/questions/659647/how-to-get-folder-path-from-file-path-with-cmd
PowerShell.exe -executionpolicy bypass -File %~dp0\scripts\hosts.ps1 %*
PAUSE
```

**hosts.ps1**

```powershell
# Get current directory
$currentDirectory = (Split-Path -Path $MyInvocation.MyCommand.Definition -Parent) + "\"
# Include inc.ps1 file
. ($currentDirectory + "inc.ps1")

# Check that we have apropriate privilegies
$principal = new-object System.Security.Principal.WindowsPrincipal([System.Security.Principal.WindowsIdentity]::GetCurrent())
$isAdmin = $principal.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
$isHostsWriteable = Test-IsWritable -Path "C:\Windows\System32\drivers\etc\hosts"
if(-not $isAdmin -And -not $isHostsWriteable) {
    Write-Host "Can not modify hosts file, run me as admin or give permission to file" -ForegroundColor red
    Exit
}

# Read config.xml
[xml]$xml = Get-Content (Join-Path -Path (Split-Path -Path $currentDirectory -Parent) -ChildPath "config.xml")

#http://technet.microsoft.com/en-us/library/ff730939.aspx
$title = "Host types to write"
$message = "What do you want to write to hosts file?"
$all = New-Object System.Management.Automation.Host.ChoiceDescription "&All", "Writes all possible hosts."
$local = New-Object System.Management.Automation.Host.ChoiceDescription "&Local", "Writes local only hosts, like: local.rabota.ua."
$remote = New-Object System.Management.Automation.Host.ChoiceDescription "&Remote", "Writes remote only hosts, like: beta.rabota.ua."
$nothing = New-Object System.Management.Automation.Host.ChoiceDescription "&Nothing", "Nothing will be written."
$options = [System.Management.Automation.Host.ChoiceDescription[]]($all, $local, $remote, $nothing)
$hostsTypeToWrite = $host.ui.PromptForChoice($title, $message, $options, 0)

if($hostsTypeToWrite -ne 3) {
    $hostsFilePath = Get-HostsFilePath

    # Remove old entries (entries that has "# IIS" at end of line)
    $hosts = Get-Content $hostsFilePath
    $hosts = $hosts | Where {$_ -notmatch '^[^#].*?# IIS'}
    $hosts | Out-File $hostsFilePath -Encoding ASCII

    $hosts = ""
    switch ($hostsTypeToWrite) {
        0 { $hosts = Get-AllProjectHosts }
        1 { $hosts = Get-LocalProjectHosts }
        2 { $hosts = Get-RemoteProjectHosts }
    }
    Write-Host $hosts -ForegroundColor DarkGray
    Add-Content $hostsFilePath $hosts
    Write-Host "Hosts file successfully updated" -ForegroundColor green
} else {
    Write-Host "Nothing to do..."
}

#Write-Host "Press any key to continue ..."
#$x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
```

IIS
---

SetupIISComponents.cmd - will install all needed iis componenets

**SetupIISComponents.cmd**

```cmd
@ECHO OFF
PowerShell.exe -executionpolicy bypass -File %~dp0\scripts\iis.ps1 %*
PAUSE
```

**iis.ps1**

```powershell
# Get current directory
$currentDirectory = (Split-Path -Path $MyInvocation.MyCommand.Definition -Parent) + "\"
# Include inc.ps1 file
. ($currentDirectory + "inc.ps1")

# Check that we have .net 4 installed
$_DOT_NET_VERSION = "v4"
$dotNet4Installed = (Get-ChildItem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP" | Where { $_.PSChildName -eq $_DOT_NET_VERSION } | Measure-Object | Select -expand Count) -eq 1
if(-not $dotNet4Installed) {
    Write-Host ".Net $_DOT_NET_VERSION is not installed run Windows Update or instal it manualy before proceed" -ForegroundColor red
    Exit
}

Write-Host "Installing IIS Components" -ForegroundColor Blue
Invoke-Expression ($currentDirectory + "pkgmgr.cmd")
Write-Host
Write-Host "Done" -ForegroundColor Green
```

**pkgmgr.cmd**

```sh
start /w pkgmgr /iu:IIS-WebServerRole;IIS-WebServer;IIS-ApplicationDevelopment;IIS-CommonHttpFeatures;IIS-HealthAndDiagnostics;IIS-Performance;IIS-Security;IIS-WebServerManagementTools;IIS-RequestFiltering;IIS-HttpCompressionStatic;IIS-HttpLogging;IIS-RequestMonitor;IIS-LoggingLibraries;IIS-HttpTracing;IIS-DefaultDocument;IIS-DirectoryBrowsing;IIS-HttpErrors;IIS-HttpRedirect;IIS-StaticContent;IIS-NetFxExtensibility;IIS-ASPNET;IIS-ISAPIExtensions;IIS-ISAPIFilter;IIS-ManagementConsole;IIS-ManagementScriptingTools
```

Sites
-----

Notice: Do not delete default site (at least one site must be present in IIS, otherwise scripts can not work - this is known problem, will hope will be fixed in next versions of IIS Web Management Scripts And Tools and\or Powershell)

**SetupSites.cmd**

```cmd
@ECHO OFF
PowerShell.exe -executionpolicy bypass -File %~dp0\scripts\sites.ps1 %*
PAUSE
```

**sites.ps1**

```powershell
$currentDirectory = (Split-Path -Path $MyInvocation.MyCommand.Definition -Parent) + "\"
. ($currentDirectory + "inc.ps1")

# Check that we have .net 4 installed
$_DOT_NET_VERSION = "v4"
$dotNet4Installed = (Get-ChildItem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP" | Where { $_.PSChildName -eq $_DOT_NET_VERSION } | Measure-Object | Select -expand Count) -eq 1
if(-not $dotNet4Installed) {
    Write-Host ".Net $_DOT_NET_VERSION is not installed run Windows Update or instal it manualy before proceed" -ForegroundColor red
    Exit
}

# Check that we have apropriate privilegies
$principal = new-object System.Security.Principal.WindowsPrincipal([System.Security.Principal.WindowsIdentity]::GetCurrent())
$isAdmin = $principal.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
if(-not $isAdmin) {
    Write-Host "Run me as admin" -ForegroundColor red
    Exit
}

Write-Host "NOTICE: Run me after IIS Components are installed" -ForegroundColor Yellow

# Include Powershell IIS snap in
Import-Module WebAdministration

Write-Host "Stopping IIS and sites..." -ForegroundColor Green
Get-Website | foreach { Stop-Website $_.name }
net stop w3svc

# Read config.xml
[xml]$xml = Get-Content (Join-Path -Path (Split-Path -Path $currentDirectory -Parent) -ChildPath "config.xml")

# Retrieve current user
$title = "Who are u?"
$message = "Chose your user name from list"
$choices = @()
$userNames = $xml.config.users.user | sort name
foreach($user in $userNames) {
    $userVariables = ($currentDirectory + "users\" + $user.name + ".ps1")
    if (Test-Path -path $userVariables) { # Show only users that have config files
        $choices += New-Object System.Management.Automation.Host.ChoiceDescription $user.name, ("Will be used in bindings like: " + $user.name + ".rabota.ua")
    }
}
$options = [System.Management.Automation.Host.ChoiceDescription[]]($choices)
$choice = $host.ui.PromptForChoice($title, $message, $options, 0)
$user = $options[$choice].Label # user will contain selected user name

# Load user specific variables
$userVariables = ($currentDirectory + "users\" + $user + ".ps1")
if (Test-Path -path $userVariables) {
    . $userVariables
}

# Check that $base path exists
if (-not (Test-Path -path $base))
{
    Write-Host "Can not access base path: $base" -ForegroundColor Red
    Exit
}

# Check that $temp path exists
if (-not (Test-Path -path $temp))
{
    # If not try to create it
    New-Item $temp -type directory

}

# Give IIS full access to temp directory
if (Test-Path -path $temp)
{
    $acl = Get-Acl $temp
    $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("IIS_IUSRS", "FullControl", "ContainerInherit, ObjectInherit", "None", "Allow")
    $acl.AddAccessRule($rule)
    Set-Acl $temp $acl
}

$base = $base.Trim().Trim("\").Trim("/") + "\"
$temp = $temp.Trim().Trim("\").Trim("/") + "\"

#if((Get-ChildItem Env:USERDOMAIN | Select -Expand Value) -ne (Get-ChildItem Env:COMPUTERNAME | Select -Expand Value)) {
#   $logonServer = Get-ChildItem Env:USERDOMAIN | Select -Expand Value
#} else {
#   $logonServer = Get-ChildItem Env:COMPUTERNAME | Select -Expand Value
#}
#$userName = Get-ChildItem Env:USERNAME | Select -Expand Value
#TODO: ask user for password to create Admin pool if it is not created

# For each project run its setup script if exists
Write-Host
foreach($project in $xml.config.projects.project)
{
    $setupScript = ($currentDirectory + "projects\" + $project.name + ".ps1")

    if (Test-Path -path $setupScript)
    {
        $site = $project.name
        . $setupScript
    }
}

Write-Host "Starting IIS and sites..." -ForegroundColor Green
net start w3svc
Get-Website | foreach { Start-Website $_.name }
```

**Example of user config file: mac.ps1**

```powershell
# This file contains user specific variables

# VARIABLES:
# ------------------------------------------------------------------------------
# $base     base path for projects, like: C:\Rabota.UA\trunk\Version\
# $temp     temp directory path for project, like: D:\Temp\

$base = "C:\Rabota.UA\trunk\Version\"
$temp = "C:\Temp"
```

**Example of rabota.ua.ps1 setup script**

```powershell
$todo = Check-Site # 0 - create, 1 - delete and create, 2 - skip

# $site - contains project name, eg: rabota.ua, channel.rabota.ua
# $user - contains current user name, eg: mac, sv, beta

# and all variables from user file

if($todo -eq 1) {
    Write-Host "Deleting $site" -ForegroundColor Blue

    Remove-Website -Name $site

    #TODO: Delete $site additional tasks here
}

if($todo -eq 0 -Or $todo -eq 1) {
    Write-Host "Creating $site" -ForegroundColor Blue

    New-WebSite -Name $site -Port 80 -HostHeader "local.rabota.ua" -PhysicalPath ($base + "Rabota2.WebUI")

    # External bindings
    New-WebBinding -Name $site -IPAddress "*" -Port 80 -HostHeader ($user + ".rabota.ua")

    # Make temporary folders
    Create-TempDir($temp + "info") # will be used to create info section apps
    Create-TempDir($temp + "HtmlTemplates") # will be used to rewrite counters
    Create-TempDir($temp + "CvPhotos")
    Create-TempDir($temp + "CvPhotos\Temp")
    Create-TempDir($temp + "Storage")
    Create-TempDir($temp + "Storage\Attaches")
    Create-TempDir($temp + "images")
    Create-TempDir($temp + "Data")
    Create-TempDir($temp + "img")

    # Make symlinks
    Create-SymLink("CvPhotos")
    Create-SymLink("Storage")
    Create-SymLink("images")
    Create-SymLink("Data")
    Create-SymLink("img")

    # Create virtual directories for assets
    New-WebVirtualDirectory -Site $site -Name CvPhotos -PhysicalPath "C:\inetpub\wwwroot\CvPhotos"
    New-WebVirtualDirectory -Site $site -Name Storage -PhysicalPath "C:\inetpub\wwwroot\Storage"
    New-WebVirtualDirectory -Site $site -Name images -PhysicalPath "C:\inetpub\wwwroot\images"
    New-WebVirtualDirectory -Site $site -Name Data -PhysicalPath "C:\inetpub\wwwroot\Data"
    New-WebVirtualDirectory -Site $site -Name img -PhysicalPath "C:\inetpub\wwwroot\img"
    New-WebVirtualDirectory -Site $site -Name HtmlTemplates -PhysicalPath ($temp + "HtmlTemplates")
    New-WebVirtualDirectory -Site $site -Name Info -PhysicalPath ($temp + "info")

    # Create Virtual apps
    New-WebApplication -Name "Info/Jobsearcher" -Site $site -PhysicalPath ($base + "Rabota2.CMS\BlogEngine.Web")
    New-WebApplication -Name "Info/Employer" -Site $site -PhysicalPath ($base + "Rabota2.CMS\BlogEngine.Web")

    # Create empty copies of HtmlTemplates files
    Get-ChildItem ($base + "Rabota2.WebUI\HtmlTemplates") | Select Name | Foreach {
        if(-not (Test-Path -Path ($temp + "HtmlTemplates\" + $_.Name))) {
            New-Item -path ($temp + "HtmlTemplates") -name $_.Name -type "file" -value ""
        }
    }

    # Move libs
    $shell_app=New-Object -com shell.application
    $filename = ($currentDirectory + "libs\ext.zip")
    $zip_file = $shell_app.namespace($filename)
    $destination = $shell_app.namespace("C:\inetpub\wwwroot")
    $destination.Copyhere($zip_file.items())
    New-WebVirtualDirectory -Site $site -Name ext -PhysicalPath "C:\inetpub\wwwroot\ext"

    #TODO: Create $site additional tasks here
}
```

Helpers
-------

**inc.ps1**

```powershell
$_HOSTS_FORMAT = "{0,-16}{1,-57}# IIS`n"

. ($currentDirectory + "Test-IsWritable.ps1")

function Get-HostsFilePath() {
    return Join-Path -path (Get-ChildItem Env:SystemRoot | Select -expand Value) -childpath "system32\drivers\etc\hosts"
}

function Get-Config() {
    [xml]$xml = Get-Content (Join-Path -Path (Split-Path -Path $currentDirectory -Parent) -ChildPath "config.xml")
    return $xml
}

function Get-LocalProjectHosts() {
    $xml = Get-Config
    $hosts = ""
    foreach($site in $xml.config.sites.site)
    {
        $hosts += $_HOSTS_FORMAT -f "127.0.0.1", ("local." + $site.domain)
    }
    return $hosts
}

function Get-RemoteProjectHosts() {
    $xml = Get-Config
    $hosts = ""
    foreach($user in $xml.config.users.user) {
        foreach($site in $xml.config.sites.site) {
            $hosts += $_HOSTS_FORMAT -f $user.ip, ($user.name + "." + $site.domain)
        }
        $hosts += "`n"
    }
    return $hosts
}

function Get-AllProjectHosts() {
    $hosts = ""
    $hosts += Get-LocalProjectHosts
    $hosts += "`n"
    $hosts += Get-RemoteProjectHosts
    return $hosts
}

function Check-Site() {
    #$siteInfo = Get-WebSite -Name $site
    $siteExists = (1 -eq (Get-WebSite | Where { $_.Name -eq $site } | Measure-Object | Select -Expand Count))

    $overwriteSite = $False
    if($siteExists) {
        $title = "Site $site already exists"
        $message = "What do you want to do?"
        $overwrite = New-Object System.Management.Automation.Host.ChoiceDescription "&Overwrite", "Site will be deleted and created again."
        $skip = New-Object System.Management.Automation.Host.ChoiceDescription "&Skip", "Leave al as is."
        $options = [System.Management.Automation.Host.ChoiceDescription[]]($overwrite, $skip)
        $chosen = $host.ui.PromptForChoice($title, $message, $options, 0)

        if($chosen -eq 0) {
            return 1
        }
    }

    if(-not $siteExists -Or $overwriteSite) {
        return 0
    } else {
        Write-Host "Site $site skipped"
        return 2
    }
}

function Create-TempDir([string]$dir) {
    # If path not exists - create it
    if (-not (Test-Path -path $dir))
    {
        New-Item $dir -type directory
    }

    # Give IIS full access to it
    if (Test-Path -path $dir) {
        $acl = Get-Acl $dir
        $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("IIS_IUSRS", "FullControl", "ContainerInherit, ObjectInherit", "None", "Allow")
        $acl.AddAccessRule($rule)
        Set-Acl $dir $acl
    }
}

function Create-SymLink([string]$dirName) {
    Invoke-Expression ("cmd /c mklink /D C:\inetpub\wwwroot\" + $dirName + " " + $temp + $dirName)
}
```

**Test-IsWritable.ps1**

```powershell
function Test-IsWritable(){
<#
    .Synopsis
        Command tests if a file is present and writable.
    .Description
        Command to test if a file is writeable. Returns true if file can be opened for write access.
    .Example
        Test-IsWritable -path $foo
                Test if file $foo is accesible for write access.
        .Example
        $bar | Test-IsWriteable
                Test if each file object in $bar is accesible for write access.
        .Parameter Path
        Psobject containing the path or object of the file to test for write access.
#>
        [CmdletBinding()]
        param([Parameter(Mandatory=$true,ValueFromPipeline=$true)][psobject]$path)

        process{
                Write-Verbose "Test if file $path is writeable"
                if (Test-Path -Path $path -PathType Leaf){
                        Write-Verbose "File is present"
                        $target = Get-Item $path -Force
                        Write-Verbose "File is readable"
                        try{
                                Write-Verbose "Trying to openwrite"
                                $writestream = $target.Openwrite()
                                Write-Verbose "Openwrite succeded"
                                $writestream.Close() | Out-Null
                                Write-Verbose "Closing file"
                                Remove-Variable -Name writestream
                                Write-Verbose "File is writable"
                                Write-Output $true
                                }
                        catch{
                                Write-Verbose "Openwrite failed"
                                Write-Verbose "File is not writable"
                                Write-Output $false
                                }
                        Write-Verbose "Tidying up"
                        Remove-Variable -Name target
                }
                else{
                        Write-Verbose "File $path does not exist or is a directory"
                        Write-Output $false
                }
        }
}
```

TODO
----

* Fill C:\inetpub\wwwroot\iisstart.htm with all sites
* Create bach file for non developers to access sites on dev machines
* Scheduled task for syncing assets
* Scheduled task for checking building results and sites availability
