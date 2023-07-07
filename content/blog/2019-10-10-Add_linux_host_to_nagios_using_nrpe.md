---
title: 'How to Add Linux Host to Nagios Monitoring Server Using NRPE Plugin'
description: "I will show you how to add remote Linux client host and it's services to Nagios Monitoring Server using NRPE agent."
image: /img/post-imgs/add-linux-to-nagios/linux2nagios_N.jpg
category: ["Monitoring", "Linux"]
type: "featured" # available types: [featured/regular]
draft: false
date: 2019-10-10
---

{{< youtube sHcWGKwRzEs >}}

In this video, I will show you how to add remote Linux client host and it's services to Nagios Monitoring Server using NRPE agent.

In my previous video, I demonstrated how to install Nagios server on Centos7. I hope you already have installed Nagios server. 

The NRPE is stands for Nagios Remote Plugin Executor. As its says by the name, NRPE allows Nagios server to discover client host resources and their services through the network.

### REMOTE CLEINT SIDE CONFIGURATION: 

##### A. Install Prerequisites
We need to install required libraries.

```bash
[root@cl1 ~]# yum install -y gcc glibc glibc-common openssl openssl-devel perl wget
```

##### B. Install Latest EPEL Repository And Update The System.
Install Latest EPEL YUM Repository

```bash
[root@cl1 ~]# wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

[root@cl1 ~]# rpm -Uvh epel-release-latest-7.noarch.rpm

[root@cl1 ~]# yum -y update
```

##### C. Install (NRPE v3) Nagios Remote Plugin Executor

```bash
[root@cl1 ~]# yum -y install nrpe
```


##### D. Update NRPE Configuration File

Add the private IP address of the Nagios server

**allowed_hosts=<Nagios_Server_IP>**

```bash
[root@cl1 ~]# vim /etc/nagios/nrpe.cfg
allowed_hosts=127.0.0.1,172.25.10.50
```


##### E. Allow NRPE service Port Through the Firewall 

Allow TCP port 5666 through the firewall to be able to listening through the TCP port 5666

```bash
[root@cl1 ~]# firewall-cmd --permanent --add-port=5666/tcp
[root@cl1 ~]# firewall-cmd --reload
```

##### F. Enable NRPE Service To Start at Boot

```bash
[root@cl1 ~]# systemctl enable nrpe.service
[root@cl1 ~]# systemctl restart nrpe.service
[root@cl1 ~]# systemctl status -l  nrpe.service
```

### NAGIOS SERVER SIDE CONFIGURATION:

##### A. Create Config a Directory

Create a directory that will store the configuration files for each server that you will need to monitor.

```bash
[root@nagios ~]# sudo mkdir /usr/local/nagios/etc/hosts
```
##### B. Create Host Definition File

Create a new configuration file for each of the remote hosts that we want to monitor.

In this case I'm going to put all the host and service definitions into a single file.


```bash
[root@nagios ~]# vim /usr/local/nagios/etc/hosts/cl1.cfg
```

```bash
# Host configuration file
define host {
        use                          linux-server
        host_name                    cl1
        alias                        Ubuntu Host
        address                      172.25.10.100
        register                     1
}

define service {
      host_name                       cl1
      service_description             PING
      check_command                   check_ping!100.0,20%!500.0,60%
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
}

define service {
      host_name                       cl1
      service_description             PING
      check_command                   check_ping!100.0,20%!500.0,60%
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       cl1
      service_description             Check Users
      check_command           check_local_users!20!50
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       cl1
      service_description             Local Disk
      check_command                   check_local_disk!20%!10%!/
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       cl1
      service_description             Check SSH
      check_command                   check_ssh
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       cl1
      service_description             Total Process
      check_command                   check_local_procs!250!400!RSZDT
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       cl1
      service_description             DNS
      check_command                   check_dns
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}
```

##### C. Modify Nagios Main Config

Then, Add "cl1.cfg" config file path into Nagios main configuration file.

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/nagios.cfg
```

```bash
# Definitions for monitoring the local (Linux) host
cfg_file=/usr/local/nagios/etc/hosts/cl1.cfg
```

##### D. Define New Commands

Edit the file commands.cfg and add the lines below.

Add this command definition config into the bottom of the "commands.cfg" file.

In this case, I'm going to add "check_dns" command into the "commands.cfg" as a new service check command.

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/objects/commands.cfg
```

Here, You can define your own commands that you want to  execute on the remote host.

You can add and modify new command definitions by editing this section.

```bash
#'check_dns' command definition
define command {
        command_name    check_dns
        command_line       $USER1$/check_dns -H $HOSTADDRESS$ -s $ARG1$ 
}
```


OR

```bash
#'check_dns' command definition
define command {
        command_name    check_dns
        command_line    $USER1$/check_dns -H digitalave.github.io -s 8.8.8.8 -t 15 | sed  -e "s/returns.*//g" 
}
```


##### E. Verify NRPE Daemon

Finally verify the configuration that we made.

```bash
/usr/local/nagios/libexec/check_nrpe -H <remote_linux_ip_address>
```

##### F. Verify Nagios Configs and Check For Error

```bash
[root@nagios ~]# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Now restart Nagios server to reflect the configuration changes.

```bash
[root@nagios ~]# systemctl restart nagios httpd
```

##### G. Log Into Nagios Web Console

