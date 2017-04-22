---
layout: post
title: "Extending Storage Spaces Volumes"
date: 2017-04-22
---

One of the cool features introduced in Windows Server 2012 was Storage Spaces, which is the ability to create Storage Pools that span multiple physical disks; it is sort of like software RAID on steroids. Storage Spaces was further enhanced in 2012 R2, and some additional polish has been added in 2016.

The most useful application of Storage Spaces in recent times is the ability to go beyond the disk size limit in public clouds such as Azure and AWS by adding multiple virtual hard drives to your VMs and creating a Storage Pool over the top. 

However, what happens when you want to dynamically extend a virtual disk created on top of a Storage Pool? Do you need to shut everything down, and potentially stop one of your business critical applications from running? Happily, no: you can dynamically expand the size of your virtual disks using the following PowerShell: 

First, create the Storage Pool. This command gets all available data disks attached to the VM and adds them to the pool:

```powershell
New-StoragePool –FriendlyName StoragePool1 -StorageSubSystemFriendlyName 'Windows Storage*' -PhysicalDisks (Get-PhysicalDisk -CanPool $True)
```

Next, create a virtual disk on top of the Storage Pool and format it to get a partition:

```powershell
$Disks = Get-StoragePool –FriendlyName StoragePool1 -IsPrimordial $False | Get-PhysicalDisk
New-VirtualDisk –FriendlyName VirtualDisk1 -ResiliencySettingName Simple –NumberOfColumns 2 –UseMaximumSize –Interleave 256KB -StoragePoolFriendlyName StoragePool1
Get-VirtualDisk –FriendlyName VirtualDisk1 | Get-Disk | Initialize-Disk –Passthru | New-Partition –AssignDriveLetter –UseMaximumSize | Format-Volume –AllocationUnitSize 64KB
```

A few important things to note about the above commands:
* ResiliencySettingName = Simple: this is equivalent to RAID 0 i.e. write the data over multiple disks, no additional copies required
* NumberOfColumns = 2: the number of disks over which to write data simulataneously. By specifying 2 here, we'll need to add new disks in multiples of 2 to be able to add them to the Pool
* Interleave = 256KB: how much data is written to each column simultaneously; adjust to suit your workload
* AllocationUnitSize = 64KB: choose the best size for your workload

I highly recommend checking out [this article](https://technet.microsoft.com/library/my243829.aspx) on Storage Spaces design to help choose the best configuration for you.

Once we have the pool configured and a volume created, we can extend it further by adding more virtual disks; remember that the number of disks you will need to add depends on the Resiliency and Column settings you chose above. 

Add some more disks to your VM without shutting it down, and they should appear inside the OS. We can add these disks to the pool and then confirm the available size for disks:

```powershell
Get-StoragePool -FriendlyName StoragePool1 | Add-PhysicalDisk -PhysicalDisks (Get-PhysicalDisk -CanPool $true)
Get-StoragePool -FriendlyName StoragePool1 | Get-VirtualDiskSupportedSize -ResiliencySettingName Simple
```

Next, we need to increase the virtual disk to take up the additional space in the pool:

```powershell
$disksize = (Get-VirtualDisk -FriendlyName VirtualDisk1).Size
$poolsize = (Get-VirtualDiskSupportedSize -StoragePoolFriendlyName StoragePool1 -ResiliencySettingName Simple).VirtualDiskSizeMax
Resize-VirtualDisk -FriendlyName VirtualDisk1 -Size ($disksize + $poolsize)
```

Finally, we can extend the required partitions to take up the available space:

```powershell
$VirtualDisk = Get-VirtualDisk VirtualDisk1
$Partition = $VirtualDisk | Get-Disk | Get-Partition | Where PartitionNumber -Eq 2
$Partition | Resize-Partition -Size ($Partition | Get-PartitionSupportedSize).SizeMax 
```

Hope this helps!