---
layout: post
title: "Update Baselines... Compliant?"
date: 2015-03-03
---

VMM has the ability to integrate with WSUS to automate the application of updates to the servers in your VMM environment. This includes Hyper-V hosts, VMM infrastructure servers, such as PXE and Library servers, and the VMM server itself. Whilst the updates view is a tad clunky, I highly recommend using VMM to update your hosts at a minimum, as VMM’s intelligent placement of VMs is much more effective than the standard Cluster Aware Updating (CAU). In addition, if VMM is integrated with SCOM, the maintenance mode settings will be passed through to SCOM to avoid any unnecessary alerts.

However, when something goes wrong with the WSUS integration, it is quite difficult to track down where the problem might lie. I had a situation where the updates were visible in VMM, and I could create a baseline and apply it to the hosts, and all hosts were reporting as Compliant. All very well and good, but upon closer inspection it seemed that certain updates were missing from the compliance report on the hosts.

When I looked at the hosts in question, the updates weren’t installed. However, when I manually installed one of the updates, it popped up in the compliance report in VMM, all the while with the host reporting as Compliant.

Nothing appeared to be out of the ordinary in VMM, so the next step was to look at the WSUS server that VMM was using. The customer had two WSUS instances with a DFS share for update storage, with one instance being used by SCCM and one by VMM; an unusual configuration, but not the root of the problem. Upon further investigation, the WSUS server in use with VMM had a lot of errors in the logs about not being able to download updates from Microsoft.

Now we were getting somewhere! The WSUS instance was configured to use a proxy server for internet access, so we tried removing that and giving the WSUS server direct access to the internet, and suddenly everything started working again. WSUS was able to download the updates, which then propagated to VMM, and then reported all the servers as Non-Compliant as expected.