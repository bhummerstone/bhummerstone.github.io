---
layout: post
title: "SCOM - Not AlwaysOn!"
date: 2015-05-12
---

I’ve been working with a customer to perform an upgrade of their entire System Center suite from 2012 SP1 to 2012 R2. Microsoft have the order and procedures for the upgrade fairly well documented on Technet, but there are a few pieces missing; I encountered one such piece when upgrading Operations Manager (SCOM).

The customer was using SQL 2012 SP1 for the Operational database, and had created an AlwaysOn Availability Group (AAG) for High Availability. The SCOM management servers were pointing at the AAG listener, which was able to failover between the SQL cluster nodes without any issues.

However, when I ran the 2012 R2 setup from the SCOM installation media on the primary management server, it seemed to get stuck at the “Verifying Prerequisites” stage. After about 10 minutes or so, I cancelled the installation, and tried on the other management server: same problem.

SCOM stores its setup log files in %LocalAppData% \SCOM\LOGS\OpsMgrSetupWizard.txt, so this was the obvious next thing to check. Upon reviewing the logs, this was the last line:

```
Info: :Info:Creating db path: \\<AAG_Listener>\M$\SCOMOP\MSSQL11.SCOMOP\MSSQL\Database
```

It seems that the SCOM install talks to SQL to determine the file path to the Database, but then attempts to contact it over a standard UNC path. As the underlying disk is not clustered, because the database replication is handled by the AAG, the above UNC path doesn’t resolve, and so the install was timing out.

To resolve this issue, I had to reconfigure both SCOM management servers and the Operational database to point to the active node of the AAG cluster so that it was no longer using the AAG listener. The best method for this is to perform steps 1, 4, 5, 6 and 8 from the “Moving the Operational Database” Technet article: <https://technet.microsoft.com/en-gb/library/hh278848.aspx>

After reconfiguring both management servers, the upgrade was able to complete without any issues. Once both servers had been upgraded, I repeated the steps above to re-point the servers back to the AAG listener.