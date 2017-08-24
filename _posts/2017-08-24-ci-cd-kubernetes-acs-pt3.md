---
layout: post
title: "CI/CD with Kubernetes on ACS - Part 3 - Configuring VSTS"
date: 2017-08-24
---

Part 1 - [Introduction] ({% post_url 2017-04-12-c1-cd-kubernetes-acs-pt1 %})  
Part 2 - [Creating ACS & ACR]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %})  
Part 3 - Configuring VSTS (this post)  
Part 4 - [Kubernetes-ifying Application]()  
Part 5 - [Build Definition]()  
Part 6 - [Release Definition]()  
Part 7 - [Wrap-up]()  


## Getting Started with VSTS
Our next step is to get our Visual Studio Team Services (VSTS) environment ready to go. If you haven't got a VSTS account already, head over to https://www.visualstudio.com/team-services/ and sign up for free.

Once you have a VSTS account, create a new project; keep the default settings for now.

The first thing we'll need to do is to authorise VSTS with GitHub so that it can access your repositories. Note that it is perfectly appropriate, and arguably preferable, to use VSTS itself to host your code, but mine is in GitHub so this step is required!

In your project, click on the little cog at the top to go to Settings, then select Services and New Service Endpoint -> GitHub. You can choose to either autorise VSTS entirely, or pre-create a Personal Access Token in GitHub for VSTS to use. I chose to "Grant Authorization", and gave the endpoint a suitable name.


## Add ACR to VSTS
Next, we'll need to create another Service Endpoint for our ACR. In the same Settings -> Services screen as before, choose New Service Endpoint -> Docker Registry.

Choose a suitably descriptive name for the connection, and enter your full ACR name in the "Docker Registry" box i.e. myacr-microsoft.azurecr.io.

I hope you noted down the appID and Password from [Part 2]({% post_url 2017-04-12-ci-cd-kubernetes-acs-pt2 %}), because you need to enter these as the Docker ID and Password respectively.


## Add SSH to VSTS
The final service endpoint we need is an SSH connection to our k8s master. Choose New Service Endpoint -> SSH to create the connection.

The Host name should be the DNS name associated with your Master nodes; in my example it is bhk8s.westeurope.cloudapp.azure.com.

The User name will be azureuser, and you can get the Private Key by SSH-ing into the k8s master and copying it from there (it will be in ~/.ssh/id_rsa by default).


## Create the VSTS Build Agent
This step is optional, but I found having a custom VSTS build agent useful to understand what was going on and for debugging purposes. In addition, depending on your requirements you may need some additional software that isn't installed on the Hosted agents, or you may burn through your free build/release time quite quickly.

Anyhow, if you do decide to create your own agent, the first step is to create a new agent pool, which can be done by going to Settings -> Agent queues -> New queue...

Once you have the queue/pool configured, you will need to create a VSTS Personal Access Token for you agent to use. The full steps are [here](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate), and you will need to assign permissions for "Agent Pools (read, manage)" for the token. Make a note of the token for later.

Finaly, it is time to set up the build agent. I used a new Ubuntu 16.04 VM (running in Azure, natch). I didn't do anything special apart from installing latest updates and [installing Docker](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-docker-ce).

You'll also need to install a few pre-reqs for the build agent:

```bash
sudo apt-get install -y libunwind8 libcurl3
```

The full steps for installing the agent are [here](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-linux). The documentation is pretty good and up-to-date, but remember to configure it to join the correct Agent Queue that we created earlier.

Now we've got VSTS sorted, the next step is to turn the existing Docker Compose file into a Kubernetes YAML; join me in [Part 4]() for that.