---
title: 'Configure Local DHCP Server and DNS Resolver on pfSense'
image: /img/post-imgs/pfsense-dns-dhcp/pf-DNS&DHCP.png
categories: ["pfSense", "Linux", "IT Security"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2016-09-18
---

{{< youtube J50FFKkkpD0 >}}

Hi Guys today I'm goning to demonstrate how to install and configure dhcp server and dns reslover on pfsense 2.4.

#### STEP 01: GENARAL CONFIGURATION

Systemc > Genaral Setup

Goto "System" tab and select "Genaral Setup" from the drop down menu.

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_01.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Change following fields as seen below.

Hostname : Define a meaningfull hostname for pfSense.

Domain Name : Define your domain name which pfsense router used.

DNS Server : Define public authoritative DNS servers for user pfSense itself.

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_02.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

#### STEP 02: SETUP DHCP SERVER

Goto  Services tab and select DHCP Server from  the drop down menu.

Services > DHCP Server

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_2.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Select the interface which you want to DHCP server runing on.

Fill the relevent fields with "Subnet", "Subnet Mask", "Range", "DNS Servers", "Gateway" and "Domain Name"

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_3.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_4.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_5.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Change all other options according to your requirement.

#### STEP 02: SETUP DNS SERVER

Unbound is integrated into pfSense. Unbound is use as the DNS server. 

Go to  "Services" tab and select "DNS Resolver" from the drop down menu.

Services > DNS Resolver

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_6.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
Enable DNS Resolver: Enable/Disable DNS Resolver

Network Interfaces: Network interfaces which are listening from  DNS queries from  clients.

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_7.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Outgoing Network Interfaces: Specific interfaces which are passing outbound DNS queries to higer level DNS server such as Google DNS. Most cases WAN interfaces used.

Enable DNSSEC Support: DNSSEC provides added security to DNS queries which validate DNS queries.

Enable Forwarding Mode: Unbound DNS queries forwarding to upstream DNS server which are defined under  System > General 

Register DHCP leases in the DNS Resolver: DHCP static mappings can be registered in Unbound which enables the resolving of hostnames that have been assigned addresses by the DHCP server in pfSense

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_8.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Host Overrides: Allows creation of custom DNS responses/records to create new entries that do not exist in DNS outside the firewall, or to override DNS responses for other hosts.

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_9.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Domain Override: For domains that should be queried by a specific remote server.
At this point I want to redirect all DNS quest for  .youtube.com and .facebook.com redirect to localhost/127.0.0.1 itself.

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_10.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_11.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

#### STEP 02: Verifing DNS Queries.

{{< image src="/img/post-imgs/pfsense-dns-dhcp/Screenshot_12.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
