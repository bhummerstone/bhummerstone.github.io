---
layout: post
title: "AddinPipeline Error in VMM"
date: 2015-01-26
---

When doing a new install of VMM, I encountered the following error when opening the console after applying Update Rollup (UR) 3:

```
Could not update managed code add-in pipeline due to the following error:
Access to the path ‘C:\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\Bin\AddInPipeline\PipelineSegments.store’ is denied.
```

After checking out the permissions on the AddInPipeline folder, it appeared that my account didn’t have the necessary permissions to access it.

Apparently, this is a known issue following UR1, but I’ve seen it on clean installations with UR3 as well. Haven’t seen it yet with UR4, so hopefully it has been resolved once and for all!

However, if you are one of the unlucky ones to encounter this, then follow these steps to resolve the issue:
1. Browse to C:\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin
2. Right-click the AddInPipeline folder, and then click Properties.
3. On the Security tab, click Advanced, and then click Continue.
4. Select the BUILTIN group, and then click Edit.
5. Click the Select a principal link, type Authenticated Users, and then click OK
6. Click OK to close the properties windows

This restores the “read and execute” permissions to the built-in Authenticated Users security group.

See the following KB article for more information: <http://support.microsoft.com/kb/2904712>