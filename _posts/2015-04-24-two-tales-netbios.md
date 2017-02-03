---
layout: post
title: "Two Tales of NETBIOS"
date: 2015-04-24
---

**Tale 1 – VMM**

VMM can build Hyper-V hosts using a process called Bare-Metal Provisioning, which takes a bare-metal server, pulls down a sysprepped VHDX file, and then enables Hyper-V and adds it to the VMM infrastructure. VMM integrates with Windows Deployment Services (WDS) to automatically boot the host from the network using PXE to download a customised Windows PE image containing the build task sequence. This is the reason that elements of the Windows ADK are pre-requisites for the VMM install, as they are used to create the custom WinPE boot image.

I faced a strange issue at a customer site recently where the WDS server was already installed and was acting as a distribution point for SCCM. This is definitely a supported configuration as long as the WDS server is added to VMM after SCCM has already been installed and configured.

However, when trying to add the WDS server to VMM, we received the following error:

```
Error (2912)
An internal error has occurred trying to contact the NO_PARAM server: NO_PARAM: NO_PARAM.
NO_PARAM
Element not found (0x80070490)
```

Helpful. No errors in the event log, nothing on the PXE server. Running the corresponding command from PowerShell with the –Verbose switch shed no extra light on the situation. This one almost had us stumped, but the error message is actually more helpful than it appears.

Because there isn’t a server name displayed, it is not being passed through correctly, or cannot be resolved. After digging around in the network settings (and a fair bit of Googling for similar messages), we discovered that NETBIOS was set to the default value of being disabled if using DHCP, but enabled if using static IPs. We changed this setting to explicitly enable NETBIOS, and the PXE server was then able to be added successfully.


**Tale 2 – SCCM**

Another customer was using SCCM for OS deployments, with a few task sequences for the various different OSs.
One of the steps in the task sequence was to join the computer to the domain. In SCCM, the settings for this are defined in the “Apply Windows Settings” step, which is then used to generate a custom unattend.xml file that is passed to the OS. These settings include the user and company name, along with any domain settings such as the domain to join, the OU to place the computer object in, and the user account to use for the operations.

In the task sequence for Windows 7 devices, the domain join step was failing. There was nothing untoward in the SMSTS.log file on the client machine, and domain join was working fine for the Windows 8 machines. I compared the two task sequences, and whilst there were a few minor differences, there wasn’t anything obviously untoward.

I decided to delve a little deeper into the log files on the client. Windows provides a few log files related to domain join:
* C:\Windows\debug\NetSetup.log
* C:\Windows\Panther\UnattendGC\setupact.log
* C:\Windows\Panther\UnattendGC\setuperr.log

I reviewed these logs, and discovered that the client was failing to resolve the account name for the domain, and so was failing to join. After re-reviewing the task sequences, and with the previous NETBIOS issues in mind, I found that the Windows 8 task sequence had the domain join account in the format <domain_fqdn>\\<djoin_acct>, but the Windows 7 task sequence had the account in the format <domain_NETBIOS>\\<djoin_act>. Changing the account to use the FQDN allowed the domain join process to complete successfully.

*Update (04/06):* The NetSetup.log is actually in C:\Windows\debug