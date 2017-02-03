---
layout: post
title: "NVGRE Routing"
date: 2015-04-12
---

Network Virtualisation using Generic Route Encapsulation (NVGRE) is one of the main network virtualisation technologies in the industry today. Championed primarily by Microsoft, it utilises GRE tunnels to encapsulate network packets from VMs and thus segregate traffic between virtual networks. It is one of the key components of Hyper-V Network Virtualisation (HNV), along with the Hyper-V virtual switch; the packets are encapsulated on egress through the virtual switch on a “compute” host, and de-encapsulated on ingress into another virtual switch on a “gateway” host. The traffic is then routed accordingly by a VM, or cluster of VMs, sitting on this gateway host. The virtual networks are known as “Customer” networks, and the encapsulation network is known as the “Provider” network.

One of the best use cases for HNV is for service providers: using NVGRE, tenant networks can be provisioned and segregated at the network level without the need to create and define VLANs whenever a new network is created. This helps open the way for true IaaS, as tenants can create their own virtual networks at will without causing IP conflicts, as the encapsulation ensures each packet is unique.

I was implementing HNV for a service provider recently, and they wanted to use different VLANs for the compute cluster and gateway clusters. This is a slightly unusual request, but they were keen on even further traffic separation and firewalling to adhere to strict security standards.

When I first created the logical networking in VMM, I created separate network sites for the compute and gateway clusters, and corresponding IP pools to ensure that IPs were automatically assigned to the encapsulation network when a VM is placed on a host. However, after creating a VM on the compute cluster, it didn’t seem to be able to access the internet.

I started by logging onto the compute hosts and trying to see if I could ping the default gateway for the Provider network. Note that Microsoft has introduced a new “-p” switch to ping that forces the ping traffic to use the Provider network. This ping was successful. Also, using the PowerShell command `Get-NetVirtualizationProviderAddress` showed that the host had an IP in the Provider network. I then tried pinging between the hosts in the compute cluster, and they were able to communicate as well. Finally, I tried pinging the gateway hosts, but received a “General Transmit Failure” error message. Curious indeed!

When digging through the possible PowerShell commands, I found a command called `Get-NetVirtualizationProviderRoute` but running this showed nothing for the hosts. I was able to use `New-NetVirtualizationProviderRoute` to create a new route in both networks, and everything suddenly started working. However, upon live migrating a VM from one host to another, the route I had created had disappeared!

The key element here is that the HNV network has a separate routing table to the standard network stack. As such, it wasn’t aware of the default gateway that had been assigned to the management vNIC on the hosts, and wasn’t able to route the NVGRE traffic between the two VLANs. I edited the logical network in VMM to add a default gateway to the encapsulation network, and the VMs could then access the internet as expected, even after being migrated between hosts. Success!