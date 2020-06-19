---
layout: post
title: "Automation in Action"
date: 2015-01-19
---

Note: I orginally published this post on [OCSL's blog](http://ocsl.co.uk/news-events/blog/?p=415), but am re-creating here for posterity 

One of the key tenets of Private, Public and Hybrid Clouds is the idea of service automation. This isn’t about replacing IT staff with software, it is about allowing IT staff to focus on the important things (such as the world cup) whilst your environment ticks along in the background.

An ideal starting place for automation is the provision of virtual machines (VMs). How often have you had to manually deploy a VM from a template? How about configuring an IP address after the VM has been deployed? Notifying the requestor that their VM has been provisioned? Imagine how much better it would be if this whole process was automated from start to finish.

One of our customers has done just that. Users request new virtual (or physical) machines using an Excel spreadsheet, which is then placed into a specific folder. Using System Centre Orchestrator (SCOrch) to monitor this folder, a series of runbooks are set into motion that perform the following actions:
1. Extract the information from the spreadsheet for the machines to be provisioned
2. Provision new VMs using the required specification
3. Insert new rows into the Microsoft Deployment Toolkit (MDT) database with the computer information, including IP address, computer name, OU and Task Sequence ID
4. Boot the VMs from an MDT boot ISO. If the machine is a physical box, then the ISO is attached using BMC commands e.g. HP iLO
5. The machines then run through the specified MDT task sequence, which installs and configures the OS, contacts a WSUS server for updates, and silently installs required applications and drivers
6. During the build process, the task sequence invokes an SCOrch runbook to email the IT department with the current build phase
7. As the final step of the task sequence, a separate SCOrch runbook is invoked to email the original requesting user (and the IT team) that the build has completed successfully

![Workflow overview](/images/automation-in-action.jpg)

One of the great things about this process is that it works equally well for both physical and virtual machines. In addition, it works for any virtual environment: VMware, Citrix and Microsoft all provide PowerShell modules for remote management of virtual hosts, so programmatically creating and managing VMs has never been easier.

What could improve this process would be the integration of System Centre Service Manager. This allows the creation of items called “Request Offerings” and “Service Offerings”, which are pre-packaged services available to end users. So instead of monitoring a folder for spreadsheets, a service offering could be used to collect the required information using a custom interface, and then kick off a runbook to perform all of the required steps.

I hope that this goes some way to showing the power of automation in the Cloud, and how the tight integration between the System Centre suite of products could streamline a wide variety of existing processes. 