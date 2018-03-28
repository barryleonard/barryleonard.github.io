---
layout: post
title:  "Case Of The Unexplained - Slow RuboCop Execution On Windows 10"
date:   2018-03-28 13:00:00
description: How I used ProcMon to solve slow running RuboCop
categories:
- Root Cause Fixes
permalink: 2018-03-28-case-of-the-unexplained-slow-rubocop
---

This blog post explains how I solved an issue where RuboCop would take over 10 minutes to execute against a small number of files on Windows 10. It also caused Visual Studio code to lockup.

___

### Introduction

For the past few weeks I've been seeing an issue on my Windows 10 laptop where executing RuboCop would take upwards of 10 minutes and drive the CPU up to 100% usage. This was having a knock-on effect in Visual Studio Code where it would regularly lockup.

### Reproducing The Issue

I used the following command to re-create the issue:

{% highlight PowerShell %}

cd c:\code\chef-repo\cookbooks\barry-example-windows-cookbook

chef exec rubocop . 

{% endhighlight %}

![placeholder](/assets/images/2018-03-28-highcpu.png "High CPU")

I could see in the processes tab in task manager that it was a Ruby process which was using 100% of the CPU so I fired up ProcMon and filtered on that process:

![placeholder](/assets/images/2018-03-28-procmon.png "Filtered ProcMon")

What I could see from the ProcMon output was that the Ruby process was iterating over thousands of files in folders under the directory C:\Users\bleonard\.cache\rubocop_cache

### Fixing The Issue

I examined the folder structure and determined that these files were temp files which were safe to delete:

![placeholder](/assets/images/2018-03-28-cache.png "RuboCop Cache")

Once I had deleted the cache, re-launched PowerShell and ran RuboCop it took about 15 seconds to complete which is the time I would expect this operation to take. I also found that Visual Studio Code was much quicker after deleting the RuboCop cache.

### Conclusion

ProcMon is a really powerful troubleshooting tool and I would recommend that people try it out when trying to solve issues such as this. It's also important to have a scheduled task to clean out the RuboCop cache periodically.
