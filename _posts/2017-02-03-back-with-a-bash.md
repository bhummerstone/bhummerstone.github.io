---
layout: post
title: "Back with a Bash!"
date: 2017-02-03
---

Been a hectic past year, but I'll hopefully be updating this blog a bit more frequently now.

As a predominantly Windows guy, one of the most interesting features in Windows 10 is the introduction of the "Bash on Ubuntu on Windows" shell that you can use. 
I've been trying to learn more Linux as part of studying for the LFCS exam, so this seemed like a great opportunity to get to grips with it.  

Installation can be a bit tricksy, but these are the rough steps. Note you will need the Anniversary update for Windows 10:

1. Enable the "Windows Subsystem for Linux" feature, then reboot: ![WSL Windows Feature]({{ site.url }}/assets/wsl-feature.PNG)

2. Switch on Developer mode in Settings -> Update and Security -> Developer mode, then reboot: ![Enable Developer mode]({{ site.url }}/assets/developer-mode.PNG)

3. Open a command prompt as administrator, and run the command "bash". You'll need to create a username, and you're all set!

There are a few things that don't quite work yet, and this is very much a beta release, but it is a pretty good Bash experience; no need for Putty anymore, at least!

Next time, I'll talk about some adventures with the Azure CLI.