---
layout: post
title: "Hyper-V Cluster IPs in VMM"
date: 2015-03-19
---

When refreshing a Hyper-V cluster in VMM, you receive the following error in the Jobs window:

![Hyper-V Error Message]({{ site.url}}/images/hyperv-cluster-ips.png)

The reason for this error is because the Virtual IP (VIP) of the Hyper-V cluster has been assigned to something from a VMM IP Pool. The best way to deal with this situation is to open up the VMM PowerShell console, and run the following command:

``` PowerShell
Get-SCIPAddress | ? {$_.Name -eq "cluster IP"} | Revoke-SCIPAddress
```

This will release the IP reservation from the pool. However, to prevent this error from re-occurring the next time you refresh the cluster, edit the IP pool to add the cluster IP address to the list of reserved IPs.