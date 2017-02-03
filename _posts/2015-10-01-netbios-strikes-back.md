---
layout: post
title: "NETBIOS Strikes Back"
date: 2015-10-01
---

After all the fun of dealing with NETBIOS in a previous Field Notes post, you would have thought I would have learnt my lesson: always check NETBIOS name resolution when working with System Center products! Alas, I encountered another head-scratcher, and (spoiler alert!) NETBIOS was once again the culprit.

The situation this time was adding a Hyper-V cluster in an untrusted domain in VMM. This is fully supported as long as the VMM traffic doesn’t go through a firewall (side note: expect another blog post on this in the future).

When trying to add one of the hosts in VMM, I was getting a DNS resolution error, even though I was entering the FQDN of the cluster. Conditional forwarders were configured in each domain to forward DNS queries to the correct servers, and I was able to ping the cluster name from the command line.

However, I remembered the NETBIOS problems from last time, and realised that I couldn’t ping the NETBIOS name of the cluster, or the hosts, in the other domain. I added the second domain as a custom DNS suffix in the VMM server’s network adapter, and it was able to resolve the names and add the cluster successfully.

You can configure the DNS suffix search list using the GUI, via a GPO, or by using the following PowerShell command:

``` PowerShell
Set-DNSClientGlobalSetting –SuffixSearchList “domain1.co.uk”,”domain2.co.uk”
```