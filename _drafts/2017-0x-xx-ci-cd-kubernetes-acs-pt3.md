---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 3 - Configuring VSTS"
date: 2017-04-xx
---

Part 1 - [Introduction] ({% post_url 2017-04-12-c1-cd-kubernetes-acs-pt1 %})  
Part 2 - [Creating ACS & ACR]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})  
Part 3 - Configuring VSTS (this post)  
Part 4 - [Kubernetes-ifying Application]()  
Part 5 - [Build Definition]()  
Part 6 - [Release Definition]()  
Part 7 - [Wrap-up]()  


Install Docker into VSTS
https://marketplace.visualstudio.com/items?itemName=ms-vscs-rm.docker
Create new project - K8S CI CD
Settings -> Services -> New Service Endpoint -> Github
Authorised to my GitHub


Add ACR to VSTS
Settings -> Services -> New Service Endpoint -> Docker Registry
Bhacr-microsoft.azurecr.io as Docker registry
appID for username, password, no email


Add SSH to VSTS
Settings -> Services -> New Service Endpoint -> SSH
K8S endpoint as host name (bhk8s.westeurope.cloudapp.azure.com)
Azureuser as username
Private key from command line (in .ssh/id_rsa)


VSTS Build Agent
Settings -> Agent Queues -> New Queue
	- K8scicd
Created new VM - Ubuntu 16.04
Installed Docker
Create new VSTS Personal access token
	- Agent Pools (read, manage)
	- 6ysaq4bgvgrlpk2srtcs5soiyfb33btz3h52yi2pp3ov3m5pvhjq

Installed agent as per https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-linux