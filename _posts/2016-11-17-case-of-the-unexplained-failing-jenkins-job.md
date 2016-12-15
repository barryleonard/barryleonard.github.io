---
layout: post
title:  "Case Of The Unexplained - The Failing Jenkins Job"
date:   2016-11-17 13:30:00
description: This is a quick blog post for an RCA conducted by one of my colleagues for a failing Jenkins job
categories:
- Root Cause Fixes
permalink: 2016-11-17-case-of-the-unexplained-failing-jenkins-job
---

An amusing tail of what the Cloud2Butt chrome extension can do
___

### Introduction

I'm sure it won't come as a surprise to any of you that we use Jenkins for deployments for most of our platforms. We've moved to a model which allows teams to own and control their Jenkins jobs (at least the configuration of them) in the spirit of our devops journey.

Today we found that the job was failing to deploy. An RCA conducted by my colleagues concluded that the cause of this was something rather amusing. Someone (who shall go nameless in the spirit of blameless RCA) had the Cloud2Butt plugin installed which automatically changes any reference to the word cloud straight to butt. The jobs in question deploy to Azure and therefore have .cloudapp.net URLs. Here's the commit:

![placeholder](/assets/images/2016-11-17-plugin_cens.png "Commit")

We politely asked the developer in question to remove the plugin and correct the issue and everything was well in the world again.