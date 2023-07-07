---
title: 'How To Send Alerts From Nagios Core Using Gmail'
description: "I am going to show you how to configure Nagios for email alerts that can be associated with certain services to certain administrators."
image: /img/post-imgs/nagios-snmp/1_N.jpg
categories: ["Linux", "Monitoring"]
type: "featured" # available types: [featured/regular]
draft: false
Date: 2019-10-25
---

In my previous video, I instructed you on how to install Nagios, configure the basics, and have your network monitored. Today, I am going to show you how to configure Nagios for email alerts that can be associated with certain services to certain administrators.

Before the installation begin, We need to setup an email relay server on the Nagios server. Which use to  send an automated emails when alerts are triggered.


## STEP 01: Install & Configure Postfix Mail Server

#### 1. Install Postfix Package

```bash
[root@nagios ~]# systemctl restart postfix
```


#### 2. Create SASL Password to able to authenticate with the Gmail server

Open or create the /etc/postfix/sasl/sasl_passwd file and add the SMTP Host, username, and password information:
Store Password Inside /etc/postfix/sasl_passwd

> [smtp.gmail.com]:587 <GMAIL_ID>@gmail.com:<APP PASSWORD>


```bash
[root@nagios ~]# vim /etc/postfix/sasl/sasl_passwd
```

```bash
[smtp.gmail.com]:587 username@gmail.com:exhhwavdchsihk3l
```

#### 6. Create a HASH DB for the password

Create the hash db file for Postfix by running the postmap command:

```bash
postmap /etc/postfix/sasl/sasl_passwd
```

Compile password to db (we can safely remove previous clear-text file now)

#### 7. Set required ownership/permissions for the following files:

```bash
sudo chown root:root /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl/sasl_passwd.db
sudo chmod 0600 /etc/postfix/sasl/sasl_passwd /etc/postfix/sasl sasl_passwd.db
```

#### 8. Modify relayhost directive in /etc/postfix/main.cf to match the following example:

Define the following directives at the end of **/etc/postfix/main.cf** file:

```bash
# This tells Postfix to hand off all messages to Gmail, and never do direct delivery.
relayhost = [smtp.gmail.com]:587

# This tells Postfix to provide the username/password when Gmail asks for one.
# Enable SASL authentication
smtp_sasl_auth_enable = yes
# Disallow methods that allow anonymous authentication
smtp_sasl_security_options = noanonymous
# Location of sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
# Enable STARTTLS encryption
smtp_tls_security_level = encrypt
# Location of CA certificates
smtp_tls_security_level = verify
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
```


#### 9: Enable & Restart Postfix Service 
```bash
[root@nagios ~]# systemctl enable postfix
[root@nagios ~]# systemctl restart postfix
```

#### 10: Sending an Test Email:

```bash
echo  "Mail Body - Test Message from NAgiso Digital Avenue" | mailx -vvv -s "Subjct is Mail Sending from Digital Avenue" username@gmail.com
```


## STEP 02: Define Email Notification Commands

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/objects/commands.cfg
```

```bash
# 'notify-host-by-email' command definition
define command {

	command_name    notify-host-by-email
	command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | mailx -vvv -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
}

# 'notify-service-by-email' command definition
define command {

    command_name    notify-service-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | mailx -vvv -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
}
```

## STEP 03: Define Email Contacts and Groups

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/objects/contacts.cfg
```
Once you have that done all you will need is the email addresses you want to use for your alerts.
Define contact section and define email address to need to be get notified.

Modify **"contact_name"** and **"email"** address

```bash
define contact {

    contact_name            nagiosadmin             ; Short name of user
    use                     generic-contact         ; Inherit default values from generic-contact template (defined above)
    alias                   Nagios Admin            ; Full name of user
    email                   username1@gmail.com ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
}


############### NEW CONTACT ###################
define contact{
        contact_name                    webadmin
        use                             generic-contact
        alias                           Web Admin
        email                           username1@orelit.com
        service_notification_commands   notify-service-by-email
        host_notification_commands      notify-host-by-email
        service_notification_period     24×7
        host_notification_period        24×7
        service_notification_options    w,u,c,r,f
        host_notification_options       d,u,r,f
}
```


w = notify on warning states
c = critical states
r = recovery
f = start/stop of flapping
d = notify on down states
u = notify on unreachable states
s = notify on stopped states

You are now ready to move on. The next section will be to define a contact group. Contact groups allow you to group people together so it is easier to alert specific people to certain events.

 Each group would have a specific user (or users) associated with it who would be alerted if a problem arises.

Define email group to which the mail need to  be sent.

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/objects/contacts.cfg
```

```bash
############# NEW CONTACT GROUPS ##############
define contactgroup{
        contactgroup_name       admins
        alias                   Nagios Administrators
        members                 webadmin,mailadmin,nagiosadmin
}
###############################################
```


## STEP 04: Define Contact Details in the Host & Service Configuration Section

Once you have defined all of your groups, save that file and close it. Now you have to attach groups to services so those groups will be alerted when something is wrong with their specific service. To do this open up the file...

```bash
[root@nagios ~]# vim /usr/local/nagios/etc/hosts/cl2.cfg
```

Head over to host configuration file and define "contacts" and "contact_groups"

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

Also you can define "contacts" and "contact_groups" on  the

```bash
## SNMP Service Definiton - Uptime
define service{
use                     generic-service
host_name               cl2
service_description     System uptime
check_command           SNMP-Uptime!-C digitalavenue
contacts                username@gmail.com
}
```

## STEP 05: VERIFY NAGIOS CONFIGS AND CHECK FOR ERROR

```bash
[root@nagios ~]# /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Now restart Nagios server to reflect the configuration changes.

```bash
[root@nagios ~]# systemctl restart nagios httpd
```

Once your changes have been made, restart Nagios. You can test notifications via the Nagios web portal. Follows these steps:

Click “Services” in the left menu. Click on any failed service (yellow) in the main area.
Click “Send custom service notifications" in the right menu.
Enter a custom "comment" and click the "commit" button.
Check your inbox for your notification email. If you don’t get it, check your server logs to see if there is a system issue on the Nagios server.






