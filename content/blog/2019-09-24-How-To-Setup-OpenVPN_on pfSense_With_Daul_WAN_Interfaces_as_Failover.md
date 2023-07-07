---
title: 'How To Setup an OpenVPN on pfSense with Dual-WAN Interfaces as Fail-over'
description: "I'm going to show you how to set up your own VPN connection using with OpenVPN service on the pfSense firewall."
image: /img/post-imgs/dual-wan-pf-vpn/Dual-WAN-OVPN-pfSense_N.jpg
categories: ["pfSense", "Linux", "IT Security"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2019-09-24
---

{{< youtube -uwONmaLwwg >}}

## Introduction:

**OpenVPN is really useful to access your office when you are out of the office.**

**In this tutorial, I'm going to show you how to set up your own VPN connection using with OpenVPN service on the pfSense firewall.
But, the really special thing is what would happen if your Internet link goes down?**


**Now,  Here's the solution...**

**But, How ???** 

To achieve this task, you may need to have two WAN interfaces attached to your pfSense firewall.

You will have uninterrupted VPN access, After the completion of this Multi-WAN OpenVPN setup.

### STEP 01: SETUP OpenVPN

Now head over to the "**VPN**" tab and select "**OpenVPN**" from the drop-down.

Click & open  "**Wizard**"

{{< image src="/img/post-imgs/dual-wan-pf-vpn/2.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

1. Select "**Local User Access**" as the Authentication Backend Type.

Click next & move onto "**Create CA Certificate**" section.

Create New CA Certificate with **Descriptive Certificate Name**, **Country Code**, **State of Province**, **City** and **Organization**.

{{< image src="/img/post-imgs/dual-wan-pf-vpn/3.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Then move onto the next step.

2. Now, We have to create a new server certificate, the same as we did earlier.

{{< image src="/img/post-imgs/dual-wan-pf-vpn/4.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Move to the next step after that.

3. Now, We need to modify "**General Settings**" for the OpenVPN

* As the first option, we need to select an **interface** wish to configure.

For now, I'm going to select the WAN1 interface.

* You may not need to modify the rest of the other fields. 

{{< image src="/img/post-imgs/dual-wan-pf-vpn/5.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

But, At this time, I'm going  to  change "**Encryption algorithem**."

* Now head over to the "**Tunnel Settings**" section.

Add "**Tunnel Network**" address. This is a virtual network address that provides an IP address to the client. It helps private communication between OpenVPN server & client. Provide a network address which is not existing on your network.

* Then, Add the network address of your "**Local Area Network**". 

This network should be the network that you wants to access via OpenVPN.

* Simply enable "**Inter Clint Communication**" to enable the communication between other OpenVPN clients.

* You may  need to  provide "**domain name**" and "DNS server's for the "**Client Settings**" section

* As the final step, You need to  "**Configure Firewall**" to allow connection through the tunnel network.

{{< image src="/img/post-imgs/dual-wan-pf-vpn/6.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/dual-wan-pf-vpn/7.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

And click finish button.

OK, the OpenVPN Server configuration has been completed so far.

But, We need to create two firewall NAT Port forwarding rules since we are going to use 2 Wan interfaces.

### STEP 02: Create Port Forwarding Rules.

Now head on to, "**Firewall**" Tab and select "**NAT**" from the drop-down menu.

Now create two port forwarding rules for the WAN1 interface. 

Forward OpenVPN connections to the localhost, which are coming from WAN1 & WAN2 interfaces.

> For example: If there are two WANs and the OpenVPN server is running on port 1194, set the Interface to Localhost, then add two port forwards:

**WAN1 - UDP, Source *, Destination WAN1 Address port 1194, redirect target 127.0.0.1 port 1194**
**WAN2 - UDP, Source *, Destination WAN2 Address port 1194, redirect target 127.0.0.1 port 1194**

Now create the same rule for the WAN2 interface.

### STEP 03: Install "OpenVPN Client Export" package

Move to the "**System**" menu and select "**Package Manager**." 

then search  for the "**openvpn-client-export**" package

And install.

STEP 03: Create OpenVPN Users

Then move to "**System**" and then "**User Manger**" and Create an OpenVPN client user.

Create a user certificate for each user by enabling the "**Certificate**" option

### STEP 04: OpenVPN Server Modification

Finally, head on to the "**Client Export**" section in the OpenVPN.

And modify "**OpenVPN Server**" settings.

Now, Change "**Host name resolution**" into "localhost."

And move down.

And add these entries to the "**Advanced Configurations**". 

These are the Public IP addresses which, OpenVPN connection going to use.

`remote 192.168.0.2 1194 udp`

`remote 192.168.10.250 1194 udp`

OK, Now OpenVPN configuration with Dual-WAN connections is completed. Now, It works as a fail-over connection to either connection. If one link goes down, another interface comes into action.
Especially keep in mind, If one link goes down, it will take nearly 60 seconds to establish a connection with the other interface.


#### Bottom line:

**Now, I think may you learn how to set up a fail-over Multi-WAN OpenVPN server on pfSense. I hope this helps you to achieve this task.**
**If you feel it worth it, don't forget to subscribe to me for more tech videos like this.**

** Thank you & see you in the next tutorial.**