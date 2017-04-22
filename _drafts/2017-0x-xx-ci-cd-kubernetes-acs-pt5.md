---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 5 - Build Definition"
date: 2017-0x-xx
---

Part 1 - [Introduction] ({% post_url 2017-04-12-c1-cd-kubernetes-acs-pt1 %})  
Part 2 - [Creating ACS & ACR]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})  
Part 3 - [Configuring VSTS] ()  
Part 4 - [Kubernetes-ifying Application]()  
Part 5 - Build Definition (this post)  
Part 6 - [Release Definition]()  
Part 7 - [Wrap-up]()  


Create new build definition
Build & Release -> New
Empty build
Github
	- Tick CI
	- Change queue to k8scicd
Repository -> bhummerstone/MyShop
Default branch is kubernetes
Triggers -> CI and batch changes ticked
	- Make sure branch filter is set to correct branch!

Variables -> New variable
	- Acr, value of bhacr-microsoft.azurecr.io
Builds -> Add build step -> Docker
Choose ACR for registry
Action: build an image
Docker file: src/ProductsApi/Dockerfile
Build context: src/ProductsApi
Image name: $(acr)/shop/products-api:$(Build.BuildId)

Action: Push an image
Registry: ACR
Image: $(acr)/shop/products-api:$(Build.BuildId)

Repeat for all containersdocker im

Command line: bash, -c "sed -i 's/BuildNumbers/$(Build.BuildId)/g' src/myshop-deployment.yml
Publish Artifact: src/myshop-deployment.yml, drop, Server

Save!
