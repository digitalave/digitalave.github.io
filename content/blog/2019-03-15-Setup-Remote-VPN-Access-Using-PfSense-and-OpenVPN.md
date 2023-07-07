---
title: 'Setup Remote VPN Access Using PfSense and OpenVPN'
image: /img/post-imgs/pf-vpn/pf-openvpn_N.jpg
categories:  ["IT Security"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2019-03-15
---

{{< youtube b_jzLyOyHnE >}}

Pfsense is a great firewall solution. Very reliable and comes with built in VLAN and VPN support. In this tutorial I’m going to demonstrate how to setup a user authenticated OpenVPN server in PfSense.
In this guide I assume you already have a functional pfSense firewall running.

## STEP 1: - Open OpenVPN Wizard

A. Create a Virtual Machine for Pfsense

{{< image src="/img/post-imgs/pf-vpn/image001_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/pf-vpn/image002_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Select OpenVPN Authentication Backed Type

{{< image src="/img/post-imgs/pf-vpn/image003_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

In this tutorial I have used “Local User Access” as the authenticated backed type.

## STEP 2:- Create New CA

Create a Certificate Authority to generate certificates for the OpenVPN server.

Fill out the following fields to create a new CA.

{{< image src="/img/post-imgs/pf-vpn/image004_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

## STEP 3:- Create Server Certificate

Create a Server Certificate from the CA for OpenVPN.

{{< image src="/img/post-imgs/pf-vpn/image005_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

## STEP 4:- OpenVPN Genaral Settings Configuration

In this case OpenVPN interface will listen on external facing WAN interface which is connected to the internet.

Interface: WAN

Protocol: UDP on IPv4 Only

LocalPort: 1194

Description: VPN

<img src="/assets/img/post-imgs/pf-vpn/image006_N.jpg" width="auto" alt="Digital Avenue DevOps Tutorials">
{{< image src="/img/post-imgs/pf-vpn/image006_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}


Cryptographic Settings Configuration

This section can be left default or change it upon your security needs.

{{< image src="/img/post-imgs/pf-vpn/image007.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

## STEP 5:- OpenVPN Tunnel Configuration

There are two important sections.

Tunnel Network

The tunnel networl should be a new network that does not currently exist on the network or the Pfsense firewall routing table.

When client connect to the VPN they will receive an address in this network.

Ex: 172.25.0.10/24

Local Network

Enter the network address of that client will connect to local network. Network address that Pfsense box resides.

Rest of the settings can be change according to your requirement.

{{< image src="/img/post-imgs/pf-vpn/image008.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}


## STEP 6:- OpenVPN Client Settings

The settings in the client settings section will be assigned to OpenVPN clients when they connect to the network.

If you are also using pfSense as your local DNS server, you would enter them here. Separate DNS servers also can enter here.

Optionally DNS, NTP server can be provided to the VPN clients from here.

{{< image src="/img/post-imgs/pf-vpn/image009.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pf-vpn/image010.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}


## STEP 7:- Firewall Rule creation for OpnVPN

Traffic from client to server: - If this section enabled, OpenVPN wizard will automatically generate the necessary firewall rules to permit the incoming connection to Pfsense OpenVPN server from clients anywhere on the internet.

Traffic from clients through VPN:- If this connection enabled, OpenVPN wizard will automatically generate firewall rules which allow traffic from clients connected to the VPN to anywhere on the local network.

{{< image src="/img/post-imgs/pf-vpn/image011.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Finally finish the wizard.

{{< image src="/img/post-imgs/pf-vpn/image012.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

## STEP 8:- Create VPN Users with Certificates

If you selected the “local user access” option during the VPN wizard then users can be added through the pfSense user manger.

{{< image src="/img/post-imgs/pf-vpn/image013_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Create new user.

{{< image src="/img/post-imgs/pf-vpn/image014.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/pf-vpn/image015.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/pf-vpn/image016.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

## STEP 9:- Install OpnVPN Client Export Package

Install OpenVPN Client Export package using Pfsense package manager.

{{< image src="/img/post-imgs/pf-vpn/image017_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pf-vpn/image018.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pf-vpn/image019_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

After the installation there will be a new tab named with “Client Export” in OpenVPN menu.

{{< image src="/img/post-imgs/pf-vpn/image020.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Modify “Hostname Resolution” field. By default this is set to the IP address of the interface running OpenVPN.

{{< image src="/img/post-imgs/pf-vpn/image021.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/pf-vpn/image022.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

After any changes made, click the “Save as default” button to store the settings.

## STEP 10:- Download the OpenVPN Client Packages.

{{< image src="/img/post-imgs/pf-vpn/image023.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Download and install OpenVPN client application.

> https://openvpn.net/index.php/open-source/downloads.html
> 
> https://swupdate.openvpn.org/community/releases/openvpn-install-2.4.6-I602.exe


Install downloaded OpenVPN profile.

{{< image src="/img/post-imgs/pf-vpn/image024_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pf-vpn/image025_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pf-vpn/image026_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
