---
layout: post
title: Hyper-V dealing with differential VMs with Powershell
tags: [hyperv, differentcial, powershell]
---

Here is a way to create any number of virtual machines in seconds.

First create your virtual machine and setup OS, updates, etc on it - it will be your base image.

Next, turn it off and delete virtual machine, all we need it is its vhdx.

**Create new VM from base image**

```powershell
New-VHD -Path 'D:\Hyper-V\Virtual Hard Disks\Server1.vhdx' -ParentPath 'D:\Hyper-V\Virtual Hard Disks\Server2016.vhdx'
New-VM -Name Server1 -Generation 2 -VHDPath "D:\Hyper-V\Virtual Hard Disks\Server1.vhdx" -SwitchName (Get-VMSwitch | Select-Object -First 1 -ExpandProperty Name)
Set-VMMemory -VMName Server1 -DynamicMemoryEnabled $True -StartupBytes 4GB -MinimumBytes 4GB -MaximumBytes 8GB
Set-VMProcessor -VMName Server1 -Count 2
Start-VM -Name Server1
```

**Delete VM**

```powershell
Stop-VM -Name Server1
Get-VMHardDiskDrive -VMName Server1 | Remove-Item
Get-VMHardDiskDrive -VMName Server1 | Remove-VMHardDiskDrive
Remove-VM -Name Server1 -Force
```

**Notes**

* Works perfectly with Windows Server 2016, no need in syspreping image.
* For Ubuntu - use first generation virtual machines and turn off safe boot in firmware settings.
