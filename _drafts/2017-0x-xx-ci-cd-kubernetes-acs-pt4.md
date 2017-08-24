---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 4 - Kubernetes-ifying Application"
date: 2017-0x-xx
---

Part 1 - [Introduction] ({% post_url 2017-04-12-c1-cd-kubernetes-acs-pt1 %})  
Part 2 - [Creating ACS & ACR]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})  
Part 3 - [Configuring VSTS] ()  
Part 4 - Kubernetes-ifying Application (this post)  
Part 5 - [Build Definition]()  
Part 6 - [Release Definition]()  
Part 7 - [Wrap-up]()


Update MyShop to deploy to k8s
Original: https://github.com/jcorioland/MyShop/
Forked, downloaded to machine using git clone
Git checkout acs-docs
Git branch kubernetes
Created myshop-deployment.yml from docker compose file
Edited Dockerfiles to FROM microsoft/dotnet:1.0.0-preview2-sdk
	- Current setup is for older version of dotnet
Git push --set-upstream origin kubernetes

Due to different networking on k8s, need to have different ports within same pod
	- Shop: 5000
	- Products: 5001
	- Ratings: 5002
	- Recommendations: 5003

Might be better to have separate pods

Changed settings in default.conf for nginx, then Dockerfile & launchSettings.json for each API

Also changed routing config in ShopFront/Views/Home/index.cshtml to reference relative URLs
