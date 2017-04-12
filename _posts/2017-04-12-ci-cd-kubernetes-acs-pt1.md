---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 1 - Introduction"
date: 2017-04-12
---

Part 1 - Introduction (this post)  
Part 2 - [Creating ACS & ACR]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})  
Part 3 - [Configuring VSTS]()  
Part 4 - [Kubernetes-ifying Application]()  
Part 5 - [Build Definition]()  
Part 6 - [Release Definition]()  
Part 7 - [Wrap-up]()  

I read an [article](https://docs.microsoft.com/en-us/azure/container-service/container-service-docker-swarm-setup-ci-cd) recently that discussed implementing Continuous Integration & Continuous Deployment (CI/CD) using Visual Studio Team Services (VSTS) to a Docker Swarm cluster on Azure Container Service (ACS); acronyms-ahoy!

This seemed like an excellent use of all the associated technologies, but it got me thinking: how would this work with Kubernetes as the orchestrator rather than Docker Swarm?

As a relative newcomer to container technologies and orchestrators it was a bit of an arduous task, but I finally managed to get my Build and Release pipelines sorted and deploying a (very simple) ASP.NET Core application to a Kubernetes cluster hosted on ACS.

So, starting from the beginning, what you will need to follow along with this series:

* An Azure account (free trial should be sufficient if you don't have one already)
* A Visual Studio Team Services account
* A Github account
* Some knowledge of git & Linux command line
* It will be helpful to understand the basics of Kubernetes, but I'll try to explain stuff as I go along

I used the Azure CLI v2, which you can either install locally or run as a Docker container, but Azure PowerShell should work just fine as well. You can get the Azure CLI v2 from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

When you're ready to go, check out [Part 2]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})!