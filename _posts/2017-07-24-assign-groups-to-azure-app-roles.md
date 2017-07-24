---
layout: post
title:  "Assign Security Groups To Azure App Roles In New Azure Portal"
date:   2017-07-24 16:30:00
description: How to assign AD security groups to Azure app roles in the new Azure portal
categories:
- Azure
- Octopus SSO
permalink: 2017-07-24-assign-groups-to-azure-app-roles
---

This blog post explains how to assign Active Directory security groups to Azure App Roles in the new Azure portal.

___

### Introduction

We use Octopus deploy as one of the tools to deploy code. We had a requirement to enable SSO for this tool but we needed to do this using our Azure AD (which is sync'd with our on prem directory).

We followed the tutorial on the [Octopus Deploy website](https://octopus.com/docs/administration/authentication-providers/azure-ad-authentication) but got stuck at the assigning users and groups to app roles section as we don't have access to the classic Azure portal. 
There was no documentation on the Octopus Deploy or Microsoft websites on how to do this on the new Azure portal so we logged a case with Microsoft. 2 support cases later I was lucky enough to speak to Gabriel Azul on the Cloud Identity team @ Microsoft and he advised that you actually need to use the enterprise application area in the new portal to acheive this. For those that find themselves in the same situation as us see the steps below on how to fix

### Steps to fix

1. Register the application in the App Registrations section (as per the standard Octopus Deploy instructions) 
2. Go to the Azure Active Directory blade
3. Go to Enterprise Applications tab
4. In All Apps search for the specific application
5. Go to Users and Groups and grant the required role
