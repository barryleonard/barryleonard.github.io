---
layout: post
title:  "SQL Server Automation with Chef"
date:   2017-05-22 08:00:00
description: Microsoft SQL Server automation with Chef
categories:
- Devops
permalink: 2017-05-22-microsoft-sql-server-automation-with-chef
---

Explains some of the work around infrastructure as code and server automation efforts

---

### Introduction ###

Over the past 7 months I've been working in a small team to get our production engineering initiative up and running.

We use a lot of SQL server in our environments and there are many different flavours in play depending on which platform you're talking about.

We needed to find a way to stand up different versions of Microsoft SQL server on different versions of Windows in different data centre providers. 

Before automation it would take someone in the TechOps team about **2 days** to stand up one of these servers manually.

We've managed to bring that build time down to about **60 minutes** depending on server/environment attributes and it's fully automated.

The tools we're using to achieve this are:

1. `Terraform` - This is our Infrastructure as code tool of choice. We're using it to orchestrate Microsoft Azure and VMware vClouds around the globe (we work & operate with data centres in EMEA, APAC and the US)
2. `Jenkins` - This is the tool we use to execute Terraform and Chef runs 
3. `GitHub` - This is where all our source code lives (Terraform projects, Chef cookbooks)
4. `Chef` - This is our config management tool of choice. It has all the blueprints for what a server/environment should look like
5. `Chefspec` - Used to write tests of cookbooks
6. `test-kitchen` - Used to provision our environments for testing 
7. `Azure` - Used to house the environments for integration tests on the pipelines (master branches of cookbooks are protected and tests must pass before a merge)
8. `Oracle VirtualBox` - Used to provision VMs for local development of cookbooks
 
### Provisioning Infrastructure With Terraform ###

If you haven't heard of Terraform I strongly encourage you to head over to the [Terraform website](https://www.terraform.io) and take a look. I think this tool is fantastic and I'm a big fan of the work that [Hashicorp](https://www.hashicorp.com) do.
 
We're using this tool to define what our infrastructure should look like (servers, networks, load balancers, firewalls etc.) 

One of the many benefits of this tool is that we can use it against different data centre providers and there's very little difference in what we have to change in code.

Our projects are typically broken down like this:

- `<provider-name>-provider.tf` - This is normally vCloud or Azure in our case and holds the connection strings for the provider (we pull the sensitive stuff out of Jenkins secrets)

- `variables.tf` - Holds common variables such as template names, network names, Chef server URL

- `JenkinsFile` - This has the definition of our pipeline. We have different steps for PRs and master

- `init.sh` - We use this script to manage the remote state (which we store in Azure storage). Used by Linux slaves

- `init.ps1` - Same as above but a Windows version for the Windows slaves

- `<provider>-topology.tf` - This file contains our network, storage, load balancer and firewall configuration 

- `.gitignore` and `.editorconfig` so we're not checking in sensitive stuff and we're consistent across our team with regards to editors (some people use Windows and some use Mac)

- `<platform>-<servername>-vm.tf` - Each server gets it's own .tf file most of the time except in cases where we're doing count. This approach allows us to manage each server separately. 

We had a lot of challenges to overcome in the vCloud world. Managing our Terraform projects using the above structure has given us a level of flexibility which seems to work.

We make use of the Chef provisioner in Terraform and hook the server in to our production Chef server (running on Azure). 

I'll be writing about Terraform in more depth in later posts but for now I'll conclude this section by saying that with the help of Terraform we end up with a vanilla Microsoft Windows server on a provider of our choice.

### Using Chef to Install and Configure Microsoft SQL Server ###

We've developed a Chef SQL server cookbook for our needs. First thing I need to say here is that we're planning on integrating this work back in to the community cookbook when we get some time. We've spent time adding and enhancing functionality including: 

1) ISO support (our SQL media comes from VLSC)

2) Service account functionality (we're using Chef vault for secrets)

3) Cumulative update and Service pack support

4) SQL Agent support

5) Disk handler (we store things on different disks so this support allows us to bring disks up with the correct block sizes, labels and letters)

6) Configurable tempdb settings

7) Local security policy support (grant pages in memory and delegation rights)

8) Install auto-update support

9) Microsoft SQL Server 2012/2014/2016 Standard and Enterprise support (includes management studio for SQL 2016) 

10) Custom path support for data directory, log directory, backup directory, tempdb directory etc.
 
We also spent a lot of time adding unit and integration tests (which we run on Azure during pipeline execution thanks to the great work by the folks over at [kitchen-azurerm](https://github.com/test-kitchen/kitchen-azurerm "kitchen-azurerm"))

### Steps To Add A New Microsoft SQL Server ###

1) Checkout the environment Terraform project from GitHub

2) Create a new branch

3) Duplicate a `<platform>-<servername>-vm.tf` file with a unique name

4) Perform a find and replace to give the server a unique name and IP address

5) Push the new branch to GitHub and create a pull request

6) Monitor the auto build of the pull request on Jenkins (This gives you the output of `terraform plan`) and ensure it's going to do what you expect

7) Request a peer review from a team member and have them approve if things are as you both expect

8) Merge the pull request in to master and monitor the progress on Jenkins 

9) After around 15 minutes you see the output `Apply complete! Resources: 1 added, 0 changed, 0 destroyed.`

10) Wait around 45 minutes for the Chef run to complete (monitor progress via the Chef client log)

### Screenshots Of The Pipeline And The End Result ###

![Jenkins Pipeline](/assets/images/2017-05-22-jenkins-terraform.png "Jenkins Terraform Pipeline") 

![SQL 2012 Server](/assets/images/2017-05-22-sql-server.png "SQL 2012 Server")
