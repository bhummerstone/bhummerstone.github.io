---
layout: post
title: "Extending Storage Spaces Volumes"
date: 2017-xx-xx
---

#Create Storage Pool
New-StoragePool –FriendlyName StoragePool1 -StorageSubSystemFriendlyName 'Windows Storage*' -PhysicalDisks (Get-PhysicalDisk -CanPool $True)

#Create virtual disk & partition
$Disks = Get-StoragePool –FriendlyName StoragePool1 -IsPrimordial $False | Get-PhysicalDisk
New-VirtualDisk –FriendlyName VirtualDisk1 -ResiliencySettingName Simple –NumberOfColumns $Disks.Count –UseMaximumSize –Interleave 256KB -StoragePoolFriendlyName StoragePool1
Get-VirtualDisk –FriendlyName VirtualDisk1 | Get-Disk | Initialize-Disk –Passthru | New-Partition –AssignDriveLetter –UseMaximumSize | Format-Volume –AllocationUnitSize 64KB

#Add additional disks
Get-StoragePool -FriendlyName StoragePool1 | Add-PhysicalDisk -PhysicalDisks (Get-PhysicalDisk -CanPool $true)

#Resize existing virtual disk
Get-StoragePool -FriendlyName StoragePool1 | Get-VirtualDiskSupportedSize -ResiliencySettingName Simple
$disksize = (Get-VirtualDisk -FriendlyName VirtualDisk1).Size
$poolsize = (Get-VirtualDiskSupportedSize -StoragePoolFriendlyName StoragePool1 -ResiliencySettingName Simple).VirtualDiskSizeMax
Resize-VirtualDisk -FriendlyName VirtualDisk1 -Size ($disksize + $poolsize)

#Resize existing partition
$VirtualDisk = Get-VirtualDisk VirtualDisk1
$Partition = $VirtualDisk | Get-Disk | Get-Partition | Where PartitionNumber -Eq 2
$Partition | Resize-Partition -Size ($Partition | Get-PartitionSupportedSize).SizeMax 
