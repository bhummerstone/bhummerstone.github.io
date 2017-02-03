---
layout: post
title: "SCCM 2012 SP1 to R2 Upgrade Gotchas!"
date: 2015-05-18
---

Sometimes when you are doing an upgrade of a piece of software, you come across some …interesting problems. I encountered a couple such problems recently when upgrading System Center Configuration Manager (SCCM) 2012 SP1 to R2.

A bit of background: SCCM 2012 SP1 with Cumulative Update (CU) 3, 1 Primary Site server, three separate Management Points (MPs), MPs configured to use local replicas of the site database.

**Problem 1 – Client Parameters and MPs**

The first interesting (and slightly worrying!) error we encountered was a Just-In-Time debugger popping up during the upgrade on the site server. We couldn’t see anything particularly wrong, so hit “don’t debug”, and carried on. Everything seemed to complete successfully, but the debugger continued to pop up at regular intervals.

We had a look in the MPMSI.log file on the three MPs, and spotted an interesting line about kicking off ccmsetup with a specific parameter to upgrade the SCCM client agent on the servers. Following the trail to ccmsetup.log, we discovered that this parameter was being ignored, and that ccmsetup was trying to execute with some strange parameters, including PATCH=xxx.msp, which pointed to the SP1 CU3 patch.

We couldn’t work out where these parameters were coming from, but a search of the registry led us to the key `HKLM\Software\Microsoft\CCMSetup\LastSuccessfulInstallParams`, which contained the offending parameters. We backed up the key, then deleted it, and the MP was able to complete the upgrade as expected.
 

**Problem 2 – MP Replicas**

In order to perform the upgrade, we needed to disable the MP database replicas so that there was a single point from which to upgrade the site database. Firstly, the MPs were reconfigured to point to the site database, then the database replica publications/subscriptions were removed.

After the upgrade was complete, and we’d resolved the issues described above, we re-enabled publication of the database replica, re-created the subscriptions and reconfigured the MPs to point to their local database replica. As per the Microsoft documentation, we added the SQL agent account with dbowner permissions on the database, and everything seemed to be OK: all the replicas were reporting as healthy with an excellent connection.

However, we started seeing some strange behaviour whereby Windows 7 and Server 2008 R2 clients, as well as all clients in the DMZ, would intermittently not pick up any policies. Now, the DMZ clients were configured as internet-only to force them to use a specific MP (side-note: this can now be achieved using preferred MPs in 2012 R2 CU3), but everything seemed healthy with that MP, so we were a bit stumped.

I have to give credit to the customer for figuring this one out (good job, Em!): after looking in the SQL logs on the DMZ-specific MP, they were filled with lots of “login failed” messages for the NTAUTHORITY\SYSTEM account. After adding this in with dbowner permissions for all database replicas, all the clients were able to reliably pick up policies once more.