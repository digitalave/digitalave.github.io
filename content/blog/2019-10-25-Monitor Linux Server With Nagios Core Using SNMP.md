---
title: 'Monitor Linux Server With Nagios Core Using SNMP'
description: "In this article I'm going to demonstrate how to monitor Linux host clients with Nagios Core using with SNMP."
image: "/img/post-imgs/nagios-snmp/1_N.jpg"
categories: ["Monitoring", "Linux"]
Date: 2019-10-25
type: "regular" # available types: [featured/regular]
draft: false
---

{{< youtube ythjwVpuypo >}}

**Nagios is one of the most popular industry slandered enterprise class server & network monitoring solution.** 

**Nagios core is free and open source tool which allows you to monitor your entire IT infrastructure to ensure host, services and applications functioning  properly.**

In this article I'm going to demonstrate how to monitor Linux host clients with Nagios Core using with SNMP.

SNMP. Which stands for Simple Network Management Protocol..

If you have already installed and setup Nagios Core Now Its time to  continue with me.

In the previous two videos I demonstrated how to install Nagios core on CentOS 7 and Monitor Remote Hosts and Services using NRPE agent plugin.

## REMOTE CLIENT SIDE CONFIGURATION:

#### STEP 01: Install and Configure SNMP on Remote CentOS 7 Host

Now, install SNMP and SNMP-UTILLS with some dependencies.

```bash
[root@cl2 ~]# yum -y install net-snmp net-snmp-utils
```


#### STEP 02: Backup Default SNMP configuration file.

Let't make backup of "snmpd.conf" default configuration file.

```bash
[root@cl2 ~]# cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig
```


#### STEP 03: Configure "snmpd.conf" File

And do the following modifications into "snmpd.conf"

```bash
[root@cl2 ~]# vim /etc/snmp/snmpd.conf
```

Insert following text content into the newly created "snmpd.conf" file.
Change the "community name" from public to something more secure. Edit the file “/etc/snmp/snmpd.conf” using this example as a guide:

In this case I'm going to  add "digitalavenue" as my  snmp community string. By default it would be as "public"


```bash
# First, map the community name "public" into a "security name"
#       sec.name  source          community
#com2sec notConfigUser  default       public
com2sec danetwork      172.25.10.0/24  digitalavenue
com2sec danetwork      127.0.0.1       digitalavenue
####
# Second, map the security name into a group name:

#       groupName      securityModel securityName
#group   notConfigGroup v1           notConfigUser
#group   notConfigGroup v2c          notConfigUser
group   DAGROUP        v1           danetwork
group   DAGROUP        v2c          danetwork
####
# Third, create a view for us to let the group have rights to:

# Make at least  snmpwalk -v 1 localhost -c public system fast again.
#       name           incl/excl     subtree         mask(optional)
#view    systemview    included   .1.3.6.1.2.1.1
#view    systemview    included   .1.3.6.1.2.1.25.1.1
view    all           included   .1
####
# Finally, grant the group read-only access to the systemview view.

#       group          context sec.model sec.level prefix read   write  notif
#access  notConfigGroup ""      any       noauth    exact  systemview none none
access  DAGROUP        ""      any       noauth    exact  all        none none

#syscontact Root <root@localhost> (configure /etc/snmp/snmp.local.conf)
syslocation Rack01,Server Room 01,Digital Avenue, Colombo (edit /etc/snmp/snmpd.conf)
syscontact Admin digitalavenue@gmail.com (configure /etc/snmp/snmp.local.conf)
```


Also you can define physical location of your server and relevant contact email.

#### Now check weather it's working by entering this command.

```bash
snmpwalk -v2c -c digitalavenue localhost:161
```


You will get plenty of information.

#### STEP 05: Firewall Configuration

Now I'm going to allow SNMP service port through the Firewall
So, Make sure to allow UDP port “161” through your firewall/router to this SNMP service.

```bash
[root@cl1 ~]# firewall-cmd --permanent --add-port=161/udp
[root@cl1 ~]# firewall-cmd --reload
```


#### STEP 06: Restart SNMPD Service

Restart "snmpd.service" to reflect the changes
And enable the service to  start at system boots.

```bash
[root@cl1 ~]# systemctl enable  snmpd.service 

[root@cl1 ~]# systemctl restart snmpd.service
```


#### STEP 07: Test SNMP Service

Now, This command will return plenty of information in the SNMP installed host.

Replace "digitalavenue" my community string with your snmp community string that have mentioned in the "snmpd.conf" file.

```bash
[root@cl2 ~]# snmpwalk -v2c -c digitalavenue localhost:161
```


## NAGIOS SERVER SIDE CONFIGURATIONS:

#### STEP 01: Add Command Definitions

**Now I'm going to use the default Nagios SNMP monitoring plugin, check_snmp,check_snmp_storage.pl, check_snmp_storage.pl,check_snmp_load.pl, check_snmp_int.pl scripts  to uptime, Physical Memory, Partition Size, Process, CPU Load and more.**

By defult, if check you will find all nagios plugins under /usr/local/nagios/libexec/ directory.

You can customize your commands your own way. so, please take some time to check carefully for  each plugin and their syntax.


```bash
[root@nagios ~]# ./check_dns -h
Usage:
check_dns -H host [-s server] [-q type ] [-a expected-address] [-A] [-n] [-t timeout] [-w warn] [-c crit]
```


Generally most of snmp scripts comes with -H for  host IP,  -w for warning level, -c for critical level and -C for snmp community string.

And sometimes you may need to add some specific values which depends on your script that you going to use.


##### SOME EXAMPLE COMMANDS: 

**Check for Up-time**

> /usr/local/nagios/libexec/check_snmp -H 172.25.10.110 -C digitalavenue -o .1.3.6.1.2.1.1.3.0


**Running processes**

> /usr/local/nagios/libexec/check_snmp -H 172.25.10.110 -C digitalavenue -o .1.3.6.1.2.1.25.1.6.0 -w 300 -c 400


**Load Average**

/usr/local/nagios/libexec/check_snmp -H 172.25.10.110 -C digitalavenue -o .1.3.6.1.4.1.2021.10.1.3.1 -w 2.0 -c 5.0

**Logged In Users**

> /usr/local/nagios/libexec/check_snmp -H 172.25.10.110 -C digitalavenue -o .1.3.6.1.2.1.25.1.5.0 -w 5 -c 10


**Total Physical Memory Available**

> /usr/local/nagios/libexec/check_snmp -H 172.25.10.110 -C digitalavenue -o .1.3.6.1.4.1.2021.4.5.0 -w %50 -c %75


**Total Physical Memory Used**

> /usr/local/nagios/libexec/check_snmp -H 172.25.10.110 -C digitalavenue -o .1.3.6.1.4.1.2021.4.6.0 -w %75 -c %90



**Total Physical Memory Used**

> /usr/local/nagios/libexec/check_snmp -H 172.25.10.110 -C digitalavenue -o .1.3.6.1.4.1.2021.4.11.0 -w %30 -c %25


#### Optionally you can collect snmp OIDs and thier names to retrive specific data and build your own commands.

**1. Run snmpwalk to view OID name**

```bash
snmpwalk -v 1 -c digitalavenue 172.25.10.110
```


make sure to change -c with your snmp community string

**2. Pick one, ie: HOST-RESOURCES-MIB::hrStorageSize.1 (Total Physical Memory)**

**3. Use snmptranslate to get OID number**

```bash
snmptranslate -On HOST-RESOURCES-MIB::hrStorageSize.1
```


**4. Check the OID number with check_snmp manually**

```bash
/usr/local/nagios/libexec/check_snmp -H 192.168.56.1 -C digitalavenue -o .1.3.6.1.2.1.25.2.3.1.5.1
```


**5. And create your own command definition**

```bash
define command{
        command_name    check_snmp_total_memory
        command_line    $USER1$/check_snmp -H $HOSTADDRESS$ -C digitalavenue -o $ARG1$
        }
```


**6. Finally need to create a service definition**

Add values to host-name and service_description accordingly.

```bash
define service{
        use                             generic-service
        host_name                       cl2
        service_description             Total Physical Memory
        check_command                   check_snmp_total_memory!.1.3.6.1.2.1.25.2.3.1.5.1
        }
```


And one another thing to keep in mind is all these arguments should separated by ! mark.

Headover to "/usr/local/nagios/etc/objects/commands.cfg" file and define command definitons. Which will be used to check uptime, disk usage, physical memory used. 

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/objects/commands.cfg
```


```bash
######################## USER DEFINED COMMANDS #############################

########## SNMP COMMAND CONFIGURATION SECTION

## check_snmp plugin commands definitions

## SNMP Command Definition - Up time
define command{
command_name    SNMP-Uptime
command_line    $USER1$/check_snmp -o 1.3.6.1.2.1.25.1.1.0 -H $HOSTADDRESS$ $ARG1$
}

## SNMP Command Definition - Total RAM installed 
define command{
command_name    SNMP-TotalRAMInstalled
command_line    $USER1$/check_snmp -o 1.3.6.1.4.1.2021.4.5.0 -H $HOSTADDRESS$ $ARG1$
}

## SNMP Command Definition - Total Physical Memory
define command{
command_name    HOST-RESOURCES-MIB::hrStorageSize.1
command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.25.2.3.1.5.1 -H $HOSTADDRESS$ $ARG1$
}

## SNMP Command Definition - Used Physical Memory
define command{
command_name    HOST-RESOURCES-MIB::hrStorageUsed.1
command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.25.2.3.1.6.1 -H $HOSTADDRESS$ $ARG1$
}

## SNMP Command Definition - Cached Physical Memory
define command{
command_name    HOST-RESOURCES-MIB::hrStorageSize.7
command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.25.2.3.1.5.7 -H $HOSTADDRESS$ $ARG1$
}

## SNMP Command Definition - Buffered Physical Memory
define command{
command_name    HOST-RESOURCES-MIB::hrStorageDescr.6
command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.25.2.3.1.5.6 -H $HOSTADDRESS$ $ARG1$
}

# Total ROOT Partition Size 
define command{
command_name    HOST-RESOURCES-MIB::hrStorageDescr.31
command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.25.2.3.1.3.31 -H $HOSTADDRESS$ $ARG1$
}

# Used ROOT Partition Size 
define command{
command_name    HOST-RESOURCES-MIB::hrStorageUsed.31
command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.25.2.3.1.6.31 -H $HOSTADDRESS$ $ARG1$
}

# Running processes
define command{
command_name    SNMP-TotalProccessRunning
command_line    $USER1$/check_snmp -o .1.3.6.1.2.1.25.1.6.0 -H $HOSTADDRESS$ $ARG1$
}

# CPU Load 1m
define command{
command_name    snmp_load_1m
command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.10.1.3.1 -H $HOSTADDRESS$ $ARG1$
}
# CPU Load 15m
define command{
command_name    snmp_load_15m
command_line    $USER1$/check_snmp -o .1.3.6.1.4.1.2021.10.1.3.3 -H $HOSTADDRESS$ $ARG1$
}

## check_snmp_storage command definitions

## SNMP Command Definiton - DISK
define command{
command_name	check_snmp_storage
command_line	$USER1$/check_snmp_storage.pl -H $HOSTADDRESS$ $USER8$ -m $ARG1$ -w $ARG2$ -c $ARG3$ $ARG4$ -C digitalavenue
}

## check_snmp_mem command definitions

## SNMP Command Definition - Physical Memory
define command{
command_name	check_snmp_mem
command_line	$USER1$/check_snmp_storage.pl -H $HOSTADDRESS$ $USER8$ -m $ARG1$ -w $ARG2$ -c $ARG3$ $ARG4$ -C digitalavenue
}

## check_snmp_process command definitions

## SNMP Command Definition - Process
define command{
command_name	check_snmp_process
command_line	$USER1$/check_snmp_process.pl -H $HOSTADDRESS$ $USER7$ -n $ARG1$ -w $ARG2$ -c $ARG3$ $ARG4$ -C digitalavenue
}

## check_snmp_load command definitions

## SNMP Command Definiton - CPU Load
define command{
command_name	check_snmp_load
command_line	$USER1$/check_snmp_load.pl -H $HOSTADDRESS$ $USER7$ -T $ARG1$ -w $ARG2$ -c $ARG3$ $ARG4$ -C digitalavenue
}

## check_snmp_int command definitions

## SNMP Command Definition - Interface 
define command{
command_name	check_snmp_iface
command_line	$USER1$/check_snmp_int.pl -H $HOSTADDRESS$ $USER7$ -n $ARG1$ -w $ARG2$ -c $ARG3$ -C digitalavenue
}


## 'check_dns' command definitions

## SNMP Command Definition - DNS
define command {
command_name	check_dns
command_line	$USER1$/check_dns -H digitalave.github.io -s 8.8.8.8 -t 15 | sed  -e "s/returns.*//g" 
}

########## checking using NRPE/Script #############

## check_volume command definition

define command{
command_name	check_volume
command_line	$USER1$/check_volume.sh -v $ARG1$ -w $ARG2$ -c $ARG3$
}
```

#### STEP 02: CREATE CONFIG DIRECTORY

Now, I will add the corresponding service definitions to apply the above commands that I've added into commands.cfg file.

Create a directory that will store the configuration files for each server that you will need to monitor.

```bash
[root@nagios ~]# sudo mkdir /usr/local/nagios/etc/hosts
```


#### STEP 03: CREATE HOST/SERVICE DEFINITION FILE

Create a new configuration file for each of the remote hosts that we want to monitor.

In this case I’m going to put all the host and service definitions into a single file.

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/hosts/cl2.cfg
```


Now we need to add new host & service entries for each remote server you will need to monitor.

Now I create a new cl2.cfg file for remote host which contain host & service definitions all together.

Add host definition section and change "host_name" and "address" sections.

finally we need to mention each service you want to monitor.

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/hosts/cl2.cfg
```


```bash
# Host configuration Section
define host {
        use                          linux-server
        host_name                    cl2
        alias                        CentOS 7 - Server
        address                      172.25.10.100
        register                     1
        contact_groups               admins
}
```

```bash
# Service Configuration Section

########## SNMP SERVICE CONFIGURATION SECTION

## check_snmp plugin service definitions

## SNMP Service Definition - Up time
define service{
use                     generic-service
host_name               cl2
service_description     System uptime
check_command           SNMP-Uptime!-C digitalavenue
}

## SNMP Service Definition - Total RAM installed 
define service{
use                     generic-service
host_name               cl2
service_description     Total Memory Installed
check_command           SNMP-TotalRAMInstalled!-C digitalavenue
}

## SNMP Service Definition - Total Physical Memory
define service{
use                     generic-service
host_name               cl2
service_description     Total Physical Memory
check_command           HOST-RESOURCES-MIB::hrStorageSize.1!-C digitalavenue
}

## SNMP Service Definition - Used Physical Memory
define service{
use                     generic-service
host_name               cl2
service_description     Used Physical Momory
check_command           HOST-RESOURCES-MIB::hrStorageUsed.1!-C digitalavenue
}

## SNMP Service Definition - Cached Physical Memory
define service{
use                     generic-service
host_name               cl2
service_description     Cached Physical Momory
check_command           HOST-RESOURCES-MIB::hrStorageSize.7!-C digitalavenue
}

## SNMP Service Definition - Buffered Physical Memory
define service{
use                     generic-service
host_name               cl2
service_description     Buffered Physical Momory
check_command           HOST-RESOURCES-MIB::hrStorageDescr.6!-C digitalavenue
}


## SNMP Service Definition - Total Running Process
define service{
use                     generic-service
host_name               cl2
service_description     Total Running Process
check_command           SNMP-TotalProccessRunning!-C digitalavenue
}

## SNMP Service Definition - CPU Load 1M
define service{
use                     generic-service
host_name               cl2
service_description     CPU Load 1Minute
check_command           snmp_load_1m!-C digitalavenue
}

## SNMP Service Definition - CPU Load 15M
define service{
use                     generic-service
host_name               cl2
service_description     CPU Load 15Minute
check_command           snmp_load_15m!-C digitalavenue
}

# check_snmp_storage service definitions

## SNMP Service Definition - DISK ROOT
define service{
use                     generic-service
host_name               cl2
service_description     Disk Root
check_command           check_snmp_storage!"^/$"!80!90
}

# SNMP Service Definition - DISK BOOT
define service{
use                     generic-service
host_name               cl2
service_description     Disk Boot
check_command           check_snmp_storage!^/boot!80!90
}

# check_snmp_mem service definitions

# SNMP Service Definition - Memory
define service {
use                     generic-service
host_name               cl2
service_description     Memory
check_command           check_snmp_mem!Physical Memory!80!90
}

# check_snmp_process service definitions

# SNMP Service Definition - Process SNMPD
define service {
use                     generic-service
host_name               cl2
service_description     Process SNMPD
check_command           check_snmp_process!snmpd!5,100!0!-2 -m 20,30 -u 90,99
}

# SNMP Service Definition - Process HTTPD
define service {
use                     generic-service
host_name               cl2
service_description     Process HTTPD
check_command           check_snmp_process!httpd!5,100!0!-2 -m 20,30 -u 90,99
}

# SNMP Service Definition - CPU Load
define service {
use                     generic-service
host_name               cl2
service_description     CPU Load
check_command           check_snmp_load!netsc!50!90
}

# SNMP Service Definition - CPU Load
define service {
use                     generic-service
host_name               cl2
service_description     Interface Status
check_command           check_snmp_iface!enp0s3!enp0s3!100,50!0,0
}


########## checking using NRPE/Script #############

# "check_volume.sh" Service Definition - Partition Root
define service{
use               local-service
host_name         cl2
service_description     Partition ROOT
check_command           check_volume!/!75!90
}

# "check_volume.sh" Service Definition - Partition Root
define service{
use                     local-service
host_name               cl2
service_description     Partition BOOT
check_command           check_volume!/boot!75!90
}


# "check_volume.sh" Service Definition - Partition Root
define service{
use                     local-service
host_name               cl2
service_description     Partition HOME
check_command           check_volume!/home!75!90
}
```


#### STEP 04: Add Host Definition File Path Into  Nagios Main Configuration File.

```bash
[root@nagios etc]# vim /usr/local/nagios/etc/nagios.cfg
```


```bash
cfg_file=/usr/local/nagios/etc/hosts/cl2.cfg
```



#### STEP 05: VERIFY NAGIOS CONFIGS AND CHECK FOR ERROR

At last, verify Nagios configuration files for any errors. 

```bash
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```


#### STEP 06: Restart Nagios Service

Finally restart nagios server.

```bash
[root@nagios /]# systemctl restart nagios httpd
```


Now, headover to  the Nagios Web Interface. And then you will see newly added host and its service status.

**I hope, now you will have a clear understanding of how to add Linux hosts into nagios monitoring using SNMP agent.**


**If you find this video helpful, Please subscribe my channel to keep in touch with new video guides like this.**