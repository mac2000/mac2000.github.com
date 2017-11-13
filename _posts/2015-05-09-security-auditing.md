---
layout: post
title: Security Auditing
tags: [audit, log, security, secpol, auditpol]
---

Did you know that you can monitor file changes without installing any additional software?

Watching for changes inside folder with Security Audition
---------------------------------------------------------

Most sync software relaying on watching filesystem for changes to do the work.

In C# there is `FileSystemWatcher` for this kind of applications, which has `IncludeSubdirectories` property that will automatically catch changes from underlying folders.

But there is a way to monitor changes **without** any additional software written/installed on your machine.

<amp-img src="/images/win-acl/SecurityAuditing.png" alt="Security Auditing" width="1625" height="881"></amp-img>

So what we are going to do is to enable Security Auditing for changes made to any of children inside our object.

You can enable auditing any events you want, in our example we are looking just for file changes.

Here is powershell snippet to enable auditing changes:

```powershell
$Path = 'C:\Projects'
$AuditChangesRules = New-Object System.Security.AccessControl.FileSystemAuditRule('Everyone', 'Delete,DeleteSubdirectoriesAndFiles,CreateFiles,AppendData', 'none', 'none', 'Success')
$Acl = Get-Acl -Path $Path
$Acl.AddAuditRule($AuditChangesRules)
Set-Acl -Path $Path -AclObject $Acl
```

Aaron Giuoco has written good article describing this process in more details: http://giuoco.org/security/configure-file-and-registry-auditing-with-powershell/

Manage Audit Policies
---------------------

Ok, so now when we have enabled auditing where can we see logged changes?

The answer is - in Event logs, but there is one more thing you should do before you will see your logs.

By default - logging of security auditing is turned off, you can enable it with `secpol.msc`

<amp-img src="/images/win-acl/secpol.png" alt="Audit Object Access" width="793" height="563"></amp-img>

But here is what you should be aware of - if you are on Windows 8.1 (not Pro) you have no secpol snapin available :(

But there is still way to accomplish this, and it is even better: `AuditPol.exe`

Try run: `AuditPol /get /category:"Object Access"`

And you will see:

```
System audit policy
Category/Subcategory                      Setting
Object Access
    File System                             No Auditing
    Registry                                No Auditing
    Kernel Object                           No Auditing
    SAM                                     No Auditing
    Certification Services                  No Auditing
    Application Generated                   No Auditing
    Handle Manipulation                     No Auditing
    File Share                              No Auditing
    Filtering Platform Packet Drop          No Auditing
    Filtering Platform Connection           No Auditing
    Other Object Access Events              No Auditing
    Detailed File Share                     No Auditing
    Removable Storage                       No Auditing
    Central Policy Staging                  No Auditing
```

You can enable only desired auditing rather than all with command like this:

```powershell
AuditPol /set /Subcategory:"File System" /success:enable
```

From now on, you will see your security audit logs in event logs viewer.

Security log
------------

Desired events will have **4663** EventID

Each event describes which object (file, folder) was modified in `ObjectName` and what operation was performed in `AccessMask`

To see template for event you can use following powershell snippet:

```powershell
(Get-WinEvent -ListProvider Microsoft-Windows-Security-Auditing).Events | where id -eq 4663
```

Which will show you what data passed to event and what is importang which index of event data is associated with which event detail

```
Id          : 4663
Version     : 1
LogLink     : System.Diagnostics.Eventing.Reader.EventLogLink
Level       : System.Diagnostics.Eventing.Reader.EventLevel
Opcode      : System.Diagnostics.Eventing.Reader.EventOpcode
Task        : System.Diagnostics.Eventing.Reader.EventTask
Keywords    : {}
Template    : <template xmlns="http://schemas.microsoft.com/win/2004/08/events">
                <data name="SubjectUserSid" inType="win:SID" outType="xs:string"/>
                <data name="SubjectUserName" inType="win:UnicodeString" outType="xs:string"/>
                <data name="SubjectDomainName" inType="win:UnicodeString" outType="xs:string"/>
                <data name="SubjectLogonId" inType="win:HexInt64" outType="win:HexInt64"/>
                <data name="ObjectServer" inType="win:UnicodeString" outType="xs:string"/>
                <data name="ObjectType" inType="win:UnicodeString" outType="xs:string"/>
                <data name="ObjectName" inType="win:UnicodeString" outType="xs:string"/>
                <data name="HandleId" inType="win:Pointer" outType="win:HexInt64"/>
                <data name="AccessList" inType="win:UnicodeString" outType="xs:string"/>
                <data name="AccessMask" inType="win:HexInt32" outType="win:HexInt32"/>
                <data name="ProcessId" inType="win:Pointer" outType="win:HexInt64"/>
                <data name="ProcessName" inType="win:UnicodeString" outType="xs:string"/>
                <data name="ResourceAttributes" inType="win:UnicodeString" outType="xs:string"/>
                </template>

Description : An attempt was made to access an object.

                Subject:
                    Security ID:        %1
                    Account Name:        %2
                    Account Domain:        %3
                    Logon ID:        %4

                Object:
                    Object Server:        %5
                    Object Type:        %6
                    Object Name:        %7
                    Handle ID:        %8
                    Resource Attributes:    %13

                Process Information:
                    Process ID:        %11
                    Process Name:        %12

                Access Request Information:
                    Accesses:        %9
                    Access Mask:        %10
```

Ashley McGlone has written goot article describing this things in more details: http://blogs.technet.com/b/ashleymcglone/archive/2013/08/28/powershell-get-winevent-xml-madness-getting-details-from-event-logs.aspx

So, now we can query our log to see what happens like this:

```powershell
$Target = 'C:\Projects'

$EVENT_DATA_OBJECT_NAME_INDEX = 6
$EVENT_DATA_ACCESS_MASK_INDEX = 9
$EVENT_DATA_PROCESS_NAME_INDEX = 11

$ObjectNameExpression = @{
    Name = 'ObjectName'
    Expression = {
        $_.ReplacementStrings[$EVENT_DATA_OBJECT_NAME_INDEX]
    }
}

$AccessMaskExpression = @{
    Name = 'AccessMask'
    Expression = {
        $_.ReplacementStrings[$EVENT_DATA_ACCESS_MASK_INDEX]
    }
}

$ActionExpression = @{
    Name = 'Action'
    Expression = {
        $AccessMask = $_.ReplacementStrings[$EVENT_DATA_ACCESS_MASK_INDEX]

        if($AccessMask -eq '0x10000') { return 'DELETE' }
        elseif($AccessMask -eq '0x2') { return 'WRITE' }
        elseif($AccessMask -eq '0x4') { return 'APPEND' }
        elseif($AccessMask -eq '0x6') { return 'WRITE|APPEND' }
        else { return 'Unknown ' + $AccessMask }
    }
}

$ProcessNameExpression = @{
    Name = 'ProcessName'
    Expression = {
        $_.ReplacementStrings[$EVENT_DATA_PROCESS_NAME_INDEX]
    }
}

#Get-EventLog -LogName Security -InstanceId 4663 -After (Get-Date).AddHours(-2) | select TimeWritten, $ObjectNameExpression, $AccessMaskExpression, $ProcessNameExpression | where ObjectName -Like ($Target + '*') | where AccessMask -in '0x10000', '0x2', '0x4', '0x6' | ft -AutoSize

$PastHourFileSystemSecurityLogEvents = Get-EventLog -LogName Security -InstanceId 4663 -After (Get-Date).AddHours(-3)
$Projection = $PastHourFileSystemSecurityLogEvents | select TimeWritten, $ObjectNameExpression, $AccessMaskExpression, $ActionExpression, $ProcessNameExpression
$FilterDesiredObject = $Projection | where ObjectName -Like ($Target + '*')
$FilterDesiredActions = $FilterDesiredObject | where AccessMask -in '0x10000', '0x2', '0x4', '0x6'
$FilterDesiredActions | ft -AutoSize
```

Which will give you:

```
TimeWritten         ObjectName                        AccessMask Action ProcessName
-----------         ----------                        ---------- ------ -----------
09.05.2015 13:32:39 C:\Projects                       0x2        WRITE  C:\Windows\explorer.exe
09.05.2015 13:32:39 C:\Projects\foo.txt               0x10000    DELETE C:\Windows\explorer.exe
09.05.2015 12:43:23 C:\Projects                       0x2        WRITE  C:\Windows\explorer.exe
09.05.2015 12:43:23 C:\Projects\New Text Document.txt 0x10000    DELETE C:\Windows\explorer.exe
```

Take a note here, all this can be written in one line, it is just a note for me to remember how this is done.

As about event viewer you can use something like:

```powershell
$xml = @"
<QueryList>
<Query Id="0" Path="Security">
<Select Path="Security">
*[EventData[Data[@Name='ObjectServer'] and (Data='Security')]]
and
*[EventData[Data[@Name='ObjectType'] and (Data='File')]]
and
*[EventData[Data[@Name='AccessMask'] and (Data='0x10000' or Data='0x2' or Data='0x4' or Data='0x6')]]
and
*[System[(EventID=4663)]]
</Select>
</Query>
</QueryList>
"@
Get-WinEvent -FilterXml $xml
```

But unfortunatelly `starts-with` is not supported, so you can only see all security events.

What next?
----------

Imagine you have two machines: _MachineA_, _MachineB_ each of them have _Projects_ folder which you want to sync.

You can not use _Dropbox_, _OneDrive_ etc just because they **do not allow** to ignore specific folders.

You do not want use _BitTorrent Sync_ just because you do not like it.

You can: Configure watching for changes in desired folders on each machine.

Then by event or schedule check which machine has latest changes and run _robocopy_ to sync them. With robocopy you will be able to exclude folders like _node_modules_ from being synced - profit.

Note: To not get into infinite loop - while checking last changes you should filter changes made by robocopy itself, this is why we are looking at `ProcessName` in logs.
