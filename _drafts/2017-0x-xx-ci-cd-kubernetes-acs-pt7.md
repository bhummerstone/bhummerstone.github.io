---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 7 - Wrap-up"
date: 2017-0x-xx
---

Part 1 - [Introduction] ({% post_url 2017-04-12-c1-cd-kubernetes-acs-pt1 %})  
Part 2 - [Creating ACS & ACR]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})  
Part 3 - [Configuring VSTS] ()  
Part 4 - [Kubernetes-ifying Application]()  
Part 5 - [Build Definition] ()  
Part 6 - [Release Definition]()  
Part 7 - Wrap-up (this post)


Testing
Committed to MyShop/kubernetes

Had to re-test a few times
	- Crashing pods due to incorrect dotnet versions, service principle for accessing ACR

Deleting deployment doesn't remove exposed service
