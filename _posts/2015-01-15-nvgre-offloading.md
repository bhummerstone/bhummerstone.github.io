---
layout: post
title: "NVGRE Offloading"
date: 2015-01-15
---

First post! Let's get right into it.

I recently had the opportunity to work on a Proof of Concept for a hosting provider to demonstrate the Microsoft Cloud Platform, which comprises of Hyper-V, System Center and the Windows Azure Pack. As part of this, we were using Network Virtualisation using Generic Routing Encapsulation (NVGRE) to allow tenants to create their own virtual networks, and thus provide true Infrastructure-as-a-Service.

However, during my testing, I found that the VMs that were created on these virtual networks could not get access to the internet. They were able to ping IP addresses with no problem, but could not resolve any DNS names.

Upon further investigation, I was able to use the PowerShell command

``` powershell
Resolve-DNSName –Name <dns_name> –TCPOnly
``` 
to resolve the necessary IP addresses. After a bit more digging, I discovered that all UDP traffic was being dropped, and since DNS uses UDP port 53, that is why my VMs couldn’t connect to DNS. What could cause just UDP traffic to be dropped? The answer was NVGRE offloading.

It is a difficult time to be a network hardware vendor. With continuing enhancements in network virtualisation, we are starting to see a fundamental shift in how people view network infrastructure in a datacentre. In addition, we are also seeing new technologies come into play to deal with the encapsulation protocols being used, as the encapsulation tends to break a lot of the traditional NIC enhancements made over the years, such as Large Segment Offloading and Virtual Machine Queue.

One of these technologies is the ability to offload some of the processing of the encapsulation from the CPU to the network cards. This is known as Encapsulated Task Offloading in Broadcom NICs, and Virtual Network Exceleration (VNeX) for Emulex, to name a few. These new settings allows the NICs to see inside the encapsulated packets, and use the following performance enhancements:
* Large Send Offload (LSO)
* Virtual Machine Queue (VMQ)
* Transmit (Tx) checksum offload (IPv4, TCP, UDP)
* Receive (Rx) checksum offload (IPv4, TCP, UDP)

This is all very good in theory, but as with every emerging technology, there are a few teething problems, such as the UDP issue I experienced. Disabling the NVGRE offload on the NICs resolved the issue, and the VMs were able to access the internet.

I'm sure that these issues will be ironed out over time, as it is probably only a driver update that is required, but it is definitely something to bear in mind when deploying network virtualisation in your environment.