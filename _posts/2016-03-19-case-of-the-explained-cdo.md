---
layout: post
title:  "Case Of The Unexplained - CDO Email And Missing Attachments"
date:   2016-03-19 15:37:00
description: How I used ProcMon to solve a CDO Email Issue 
categories:
- Root Cause Fixes
permalink: 2016-03-19-case-of-the-unexplained-CDO-email-attachments
---

This blog post explains how I solved an issue we had on one of our production servers just recently with CDO and email attachments.

___

### Introduction

We use the CDO component along with the IIS SMTP module to send emails out to customers on one of our production platforms. Just recently we hit an issue whereby one of our email relay servers stopped sending out email that contained attachments. 

This issue caused us a few problems because the server in question is particularly rooted in to the infrastructure so we couldn't just scrap it and rebuild. We needed to find a fix ASAP.

Our dev team wrote us a test harness which we could use to send emails out and I quickly got on with debugging the issue.

My tools of choice in a situation like this are:

- Sysinternals Process Monitor
- SysInternals Process Explorer
- WinDbg

### Reproducing The Issue

I used the following code to re-create the issue:

{% highlight vb %}

Dim sSubject, sBody, sSenderName, sRecipients, sSenderEmail, sSender, sAttachments
 
sSubject = "Dummy Subject"
sBody = "This is a SendSMTPMail Unit Test Email"
 
'N.B. Recipients should be delimited by commas or semmi-colons

sRecipients = "MY EMAIL ADDRESS"
 
'sSender = "Unit Tester"
 
'N.B. Attachments should be delimited by commas or semmi-colons      
sAttachments ="C:\example.pdf"
 
' Call SendSMTPMail(sSubject, sBody,sRecipients, sSenderEmail, sSender, sAttachments)
SendSMTPMail sSubject, sBody,sRecipients, "", "", sAttachments
 
sAttachments = ""
sSubject = "SendMailX Unit Test"
sBody = "This is a 'SendMailX' Unit Test Email"
'Call SendMailX(sBody, sSubject,sRecipients)
 
sAttachments = ""
sSubject = "SendMailXy Unit Test"
sBody = "This is a 'SendMailXy' Unit Test Email"
'Call SendMailXy(sBody, sRecipients, sSubject)
 
 
sAttachments = ""
sSubject = "SendMailWithFromAdd Unit Test"
sBody = "This is a 'SendMailWithFromAdd' Unit Test Email"
 
'Call SendMailWithFromAdd(sFromAddress, sSender, sBody, sSubject, sRecipients, sAttachments)
 
 
sSubject = "SendMailWithAttachment Unit Test"
sBody = "This is a 'SendMailWithAttachment' Unit Test Email"
 
'Call SendMailWithAttachment(sBody, sSubject, sRecipients, sAttachments)
 
sAttachments = ""
sSubject = "SendMail Unit Test"
sBody = "This is a 'SendMail' Unit Test Email"
 
'Call SendMail(sBody, sSubject, sRecipients)
</script>
</job>
{% endhighlight %}

What I found when I ran this code was the following:

![placeholder](/assets/images/2016-03-19-procmon.png "Output in ProcMon")
![placeholder](/assets/images/2016-03-19-windbg.png/800x400 "Output in WinDbg")

What we can see from the outputs is that we're hitting a buffer overflow when enumerating the HKCR of the registry, in particular around the CSV extension:

{% highlight ruby %}
SUCCESS : Index: 109, Name .csv_2010_09_07_05_34_53_2010_09_08_14_47_03_2010_09_08_15_57_04_2010_09_08_17_36_51_2010_09_08_57

BUFFER OVERFLOW : Index: 110, Length: 288
{% endhighlight %}

{% highlight ruby %}
advapi32!RegEnumKeyExA+0x12f
{% endhighlight %}

### Fixing The Issue

Now that I knew this info I went straight to the registry editor on the box in question to investigate what might be going on here.

What I found in the registry were 300+ keys of:

{% highlight ruby %}
 .csv_year_month_day
{% endhighlight %}

![placeholder](/assets/images/2016-03-19-338-keys-later.png/800x400 "Keys")


I looked at what could have been causing the creation of these keys and it looks like a process we have had been crashing out and creating these keys over a number of years. 

I verified that we didn't use/need these keys and wrote an RFC to back them up and start purging them. After I had purged them out I was able to run the test harness and emails with attachments started working again.

### Conclusion

All companies have technical debt and I would say that this is one of those areas for us. There's a real drive at the moment to resolve technical debt and moving forward we'll prevent this type of issue by having configuration management in place to allow us to provision a new instance of this server in minutes and not days. We're also looking at the architecture of the process and changing it to a model whereby things are more distributed.

