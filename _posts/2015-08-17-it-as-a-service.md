---
layout: post
title: "IT as a Service"
date: 2015-08-17
---

Traditionally, IT departments have been seen as either “the people who fix the magic box I use for work” or “the people who break the magic box I use for work”. They are black holes for budgets, always under-staffed and overworked, blamed when something changes, but never praised when it all carries on serenely. IT departments are not all cloud providers, and so resource is almost always constrained in some respects, but the rest of the business is rarely aware of this fact.

Today, when people are more technically-literate than ever and smartphones are becoming the norm, it is time for users to have a bit more interaction with the IT department, including developing an understanding about what they actually do on a day-to-day basis.

One of the best changes a company can make is to start marketing the IT department as providing a service to the rest of the business. Standard practices such as transparent chargeback and clearly defined SLAs should be implemented so your “customers” feel like IT is listening and working hard for their benefit. In addition, users will start to understand the costs associated with their choices: do they really need that fancy application they saw at an event that requires eight servers to run if it will cost them £1000 per server out of their own budget?

In return, the IT department can start to focus on what services it provides to end users, and look to add value to these where possible. For example, let’s think about Identity and Access Management (IAM). The basic starting point for this is usually Active Directory, which is made up of one or more Domain Controllers. But other solutions can be added in to enhance this basic service: Certificate Services for multi-factor authentication, Federation Services for claims-aware authentication. How about hybrid identity? Azure AD is a very attractive proposition for managing identity in the cloud, and is a great on-ramp for offering cloud-based applications such as Office 365 to your users.

Whist this change in perspective is all well and good, if the technology behind it doesn’t make it easy, then no-one will have the time and resource to be able to implement this service-focussed approach. With the release of the Azure Resource Manager (ARM) and the upcoming Azure Stack, you will be able to build templates to define your services, either in the cloud, on-premise, or both. Combined with PowerShell Desired State Configuration for VM configuration and automation to reduce or eliminate manual steps, IT departments can start making real strides towards “IT as a Service”.