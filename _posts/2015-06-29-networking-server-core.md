---
layout: post
title: "Networking in Server Core"
date: 2015-06-29
---

Microsoft has made a big push for Server Core as the default installation option for servers in recent times in order to reduce the number of required patches, reduce the installation footprint, and make the OS more secure by removing unnecessary features. As a result of this, knowing how to do things in PowerShell is becoming absolutely essential and, with the introduction of Nano Server in Windows Server 2016 (which has no GUI at all!), the command line will once again be king.

To assist with the transition, Microsoft provide an initial server configuration tool, called “sconfig”. Just type this into your Server Core machine, and you will get a command line guided wizard to perform all of the basic configuration for your servers, such as setting an IP address, changing the name and domain, and configuring Windows Updates.

However, this wizard can’t help you with everything, so for that we need to delve into PowerShell. I’ve got three examples of fairly standard procedures, so let’s get started.

The first is definitely the most common: setting an IP address. You can do this in sconfig, but it is worth knowing how to do it in PowerShell so you can script it. However, somewhat counterintuitively, you need to actually create an IP address on the adapter first, and then assign it to the adapter. The process is as follows:

``` PowerShell
New-NetIPAddress -InterfaceAlias "Ethernet” -IPAddress "192.168.10.4" -DefaultGateway "192.168.10.254" -PrefixLength 24
Set-NetIPAddress -InterfaceAlias "Ethernet” -IPAddress "192.168.10.4"
```

As an aside, I would recommend that you start getting into the habit of beginning your IP addressing from x.x.x.4 because if you will potentially migrate any workloads to Azure, this will allow you to keep the same IP addresses for your VMs (Azure uses the first 3 addresses in the range).

You’ll also need to set the DNS servers, if required, which can be done using the following command:

``` PowerShell
Set-DNSClientServerAddress –InterfaceAlias “Ethernet” –ServerAddresses “192.168.10.4”,”192.168.10.5”
```

Next, a slightly more atypical operation: disabling NETBIOS. This can give some performance advantage benefits on networks that do everything using IP addresses, such as Live Migration and Cluster networks:

``` PowerShell
$adapter = Get-WMIObject Win32_NetworkAdapterConfiguration | ? {$_.Index -eq ((Get-NetAdapter –Name “Ethernet”).ifIndex)}
$adapter.SetTCPIPNetbios(2)
```

Finally, another more advanced operation: disabling DNS registration for a network adapter. This can be useful from a management perspective when dealing with multiple network adapters in a system that you don’t want in DNS e.g. Cluster networks. The command is as follows:

``` PowerShell
Set-DNSclient -InterfaceAlias "Ethernet" -RegisterThisConnectionsAddress:$false
```
PowerShell is not going away any time soon, so start small with some basic server configuration, and then think about working your way up to more advanced operations; there’s a wealth of information out there, so don’t be afraid to give it a go!