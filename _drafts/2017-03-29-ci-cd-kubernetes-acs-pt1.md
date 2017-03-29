---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 1"
date: 2017-03-29
---

I read a blog post recently (<link here>) that discussed implementing Continuous Integration & Continuous Deployment (CI/CD) using Visual Studio Team Services (VSTS) to a Docker Swarm cluster on Azure Container Service (ACS); acronyms-ahoy!

This seemed like an excellent use of all the associated technologies, but it got me thinking: how would this work with Kubernetes as the orchestrator rather than Docker Swarm?

As a relative newcomer to container technologies and orchestrators it was a bit of an arduous task, but I finally managed to get my Build and Release pipelines sorted and deploying a (very simple) ASP.NET Core application to a Kubernetes cluster hosted on ACS.

So, starting from the beginning, what you will need to follow along with this series:

- An Azure account (free trial should be sufficient if you don't have one already)
- A Visual Studio Team Services account
- Some knowledge of git & Linux command line
- It will be helpful to understand the basics of Kubernetes, but I'll try to explain stuff as I go along

I used the Azure CLI v2, which you can either install locally or run as a Docker container, but Azure PowerShell should work just fine as well.