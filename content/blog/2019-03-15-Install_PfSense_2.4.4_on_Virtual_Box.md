---
title: 'Install PfSense 2.4.4 on Virtual Box'
image: /img/post-imgs/pf-install/pfSense-Installation_N.jpg
categories: ["IT Security"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2019-03-15
---

{{< youtube SEXfqFCh3y0 >}}

# Install PfSense 2.4.4 on Virtual Box

pfSense is network firewall based on FreeBSD operating system with a custom kernel and includes free third-party packages for additional features. It provides same functionality or more of common commercial firewalls.
In this tutorial, I will install pfSense in VirtualBox it will work as a firewall for our virtual lab environment for future articles as well.

## STEP 1:- Download Pfsense ISO Image.

Download most recent stable release from https://www.pfsense.org/download/

## STEP 2: - Install Pfsense on Virtual box
A. Create a Virtual Machine for Pfsense

{{< image src="/img/post-imgs/pf-install/PF1.gif" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Follow The Guideline as seen on GIF image.

{{< image src="/img/post-imgs/pf-install/PF2.gif" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Here we have assigned two interfaces with one Bridge Interface (em0) and one Internal Interface Named (em1) LAN2.
Assigned IP as follows…

Em0 - WAN Interface - 192.168.1.100/24
Em1 – LAN Interface – 192.168.100.1/24

DHCP Range – 192.168.100.100 – 192.168.100.200

{{< image src="/img/post-imgs/pf-install/PF3.gif" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

## STEP 2: - Logging in to Web Interface
Pfsense default username and password is “admin” and “Pfsense”

Complete Pfsense initial setup wizard.

{{< image src="/img/post-imgs/pf-install/PF4.gif" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
