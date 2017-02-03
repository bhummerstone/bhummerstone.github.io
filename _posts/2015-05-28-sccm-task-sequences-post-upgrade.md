---
layout: post
title: "SCCM Task Sequences Post-Upgrade"
date: 2015-05-28
---

As you may have guessed from recent posts, I have been doing a bit of work around upgrading the System Center components for a customer from 2012 SP1 to 2012 R2. Part of doing this work in different environments is finding out all of the little niggly things that could go wrong as part of the upgrade, as well as any undocumented post-upgrade tasks to get your environment back up and running smoothly again.

Fortunately (or unfortunately, depending on your viewpoint!), I’ve had a chance to experience quite a few of these recently, but I’ll definitely know for next time to watch out for them to offer a better experience: forewarned is forearmed!
The most recent issue we experienced was running through an OS Deployment (OSD) task sequence after upgrading SCCM to 2012 R2 CU4. The PCs would apply the OS image without any problems, and would even apply a number of packages, but would consistently fail when they got to the first application. This happened regardless of whether it was a laptop, desktop or VM.

After checking through the various log files, the culprit appeared to first appear in the DataTransferService.log, which said:

```
DTS job {A8EB7EF2-C0B8-49BC-97AE-122C0B68030F} BITS job {30929439-FBF0-43D0-AA85-97319DB4BA0B} failed to download source file 
http://<management_point_fqdn>:80/SMS_MP/.sms_dcm?Id&DocumentId=GLOBAL/Platform_Settings/PROPERTIES&Hash=0A2A0D43D908F75432386E3CEE1273A9F49CFD2518E26BE2CA7FDC017FCF0A49&Compression=zlib 
to destination C:\WINDOWS\CCM\CIDownloader\Staging\{5C7A193D-4F07-4573-B914-D55E54880B46}_2.zip with error 0x80190194
```

It is important to note at this point that SCCM uses BITS to download its policies, and this is exactly what it was failing to do with the first application. We tried disabling this particular application, and it just failed on the next one instead!

This is actually a known issue in SCCM 2012, but a hotfix was released post-CU3, and was included in CU4. The CU4 client was being slipstreamed during the install, so it shouldn’t have been failing in this way.

In the end, we managed to work around this by creating a brand new task sequence, and copy/pasting all of the steps into it. I’m not sure if 2012 R2 or CU4 changes the logic of the task sequence somehow to avoid the timeout issues referenced above, but re-creating the task sequence definitely seemed to work.

Please let me know if you have seen/experienced this issue before, and what you did to resolve it.