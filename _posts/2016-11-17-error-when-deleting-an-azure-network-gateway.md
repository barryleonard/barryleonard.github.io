---
layout: post
title:  "Error When Deleting An Azure Network Gateway"
date:   2016-11-17 20:08:00
description: This is a quick RCA for an issue observed with Microsoft Azure networking
categories:
- Root Cause Fixes
permalink: 2016-11-17-error-when-deleting-an-azure-network-gateway
---

This is a quick RCA for an issue observed with Microsoft Azure networking

---

Introduction

We recently had a requirement to setup a site-to-site VPN between one of our Microsoft Azure networks and an on-prem Cisco ASA at a third party data centre. When I setup the original network gateway I accidentally put it in to routed mode (we use this for VPN tunnels between other VNETS on Azure). I needed to change the gateway to policy based routing as that is the only routing policy that the Cisco ASA supports. Unfortunately you cannot change the routing policy of a gateway once it is created and therefore you have to destroy it and re-create it. That's where I ran in to an issue. When trying to remove any elements to do with the gateway the error being returned from Azure was:

I raised a case with Microsoft support and was lucky enough to have my case handled by Chad Glenn of the Azure Networking team in the US. He ran a few diagnostics reports and could see that something wasn't quite right. The message we were getting when trying to remove the gateway was that there was a connection which needed removing - the problem was I couldn't see it in the front end. Turns out there was a sync issue between the front end and the back end. To resolve the issue we used the https://resources.azure.com website to establish that there was indeed something in the back end which we were not seeing in the front end. Armed with these details Chad asked me to run:

`Remove-AzureRmVirtualNetworkGatewayConnection -Name '<redacted>' -ResourceGroupName '<redacted>' -verbose`

(replace redacted with what you're seeing in the resource explorer

Once I'd run the above command and refreshed the Azure resource explorer I could see the connection was removed and I was then able to remove the Azure virtual network gateway and re-create. 
One word of warning is that I got a new public IP address for the new gateway even though I didn't delete the public IP resource. This makes sense because in affect I had dissociated the public IP and that freed up the address to the pool (no point in having a public IP if it's not attached to anything).

Once I'd re-created the gateway in policy mode my contact at the third party data centre managing our ASA was able to create the tunnel for us (Cheers Andy Stas!) 

