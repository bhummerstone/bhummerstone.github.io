---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 6 - Release Definition"
date: 2017-0x-xx
---

Part 1 - [Introduction] ({% post_url 2017-04-12-c1-cd-kubernetes-acs-pt1 %})  
Part 2 - [Creating ACS & ACR]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})  
Part 3 - [Configuring VSTS] ()  
Part 4 - [Kubernetes-ifying Application]()  
Part 5 - [Build Definition] ()  
Part 6 - Release Definition (this post)  
Part 7 - [Wrap-up]()


Create release definition
Releases -> New definition -> Empty
"Choose later", set agent to k8scicd
Link to an artifact source, ensure it is Build source, click Link
Triggers -> CD -> build reference

Copy file over SSH
SSH endpoint: SSH
Source: $(System.DefaultWorkingDirectory)/myshop-k8s/drop
Target: deploy

Run SSH
	- kubectl apply -f deploy/myshop-deployment.yml --record