---
title: 'Setup Postfix To Send Emails Using Gmail Relay'
description: "Send email through the Linux terminal and email automation tasks such as Nagios, Zabbix, and many more other scripts."
image: /img/post-imgs/postfix-gmail/1_N.jpg
tags: [Postfix, Gmail, CentOS 7, RHEL 7]
categories: ["Linux"]
date: 2019-10-25
type: "regular" # available types: [featured/regular]
draft: false
---

{{< youtube c1BWzUgGm6I >}}

In this tutorial I'm going to tech you how to configure an email relay server for the with your Gmail account.
Which helps you to send email through the Linux terminal and email automation tasks such as Nagios, Zabbix, and many more other scripts.

Here I'm going to set up the email server with authentication using smtp.gmail.com as a relay host in the following way

STEP 01: Generate App Password For Postfix

#### 1. First you need to have a Gmail email account.

Login to Google account from here:

Manage your account access and security settings.

Scroll down to “Password & sign-in method" and click 2-Step Verification. Then enable it.

<a href="https://myaccount.google.com/security" target="_blank">https://myaccount.google.com/security</a>

#### 2. Click the following link to Generate an App password for Postfix.

<a href="https://security.google.com/settings/security/apppasswords" target="_blank">https://security.google.com/settings/security/apppasswords</a>

#### 3. Genarate an App Passwaord

Click Select app and choose Other (provide any name) from the drop-down menu. Enter “Postfix” and click Generate . And save the password.

STEP 02: Install & Configure Postfix Mail Server

#### 1. Install Postfix Package

```bash
[root@nagios ~]# systemctl restart postfix
```

#### 2. Install SASL binaries and libraries

```bash
[root@nagios ~]# sudo yum install cyrus-sasl cyrus-sasl-lib cyrus-sasl-plain
```

#### 3. Create SASL Password to able to authenticate with the Gmail server

Open or create the /etc/postfix/sasl/sasl_passwd file and add the SMTP Host, username, and password information:
Store Password Inside /etc/postfix/sasl_passwd

```
[smtp.gmail.com]:587 <GMAIL_ID>@gmail.com:<APP PASSWORD>
```

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

Either, You can add following entries using  "postconf -e" command

```bash
postconf -e relayhost=[smtp.gmail.com]:587
postconf -e smtp_sasl_auth_enable=yes
postconf -e smtp_sasl_security_options=noanonymous
postconf -e smtp_sasl_password_maps=hash:/etc/postfix/sasl/sasl_passwd
postconf -e smtp_tls_security_level=encrypt
postconf -e smtp_tls_security_level=verify
postconf -e smtp_tls_CAfile=/etc/ssl/certs/ca-bundle.crt
```



Finally "main.cf" file will be look alike this...

```bash
alias_database = hash:/etc/aliases
alias_maps = hash:/etc/aliases
command_directory = /usr/sbin
config_directory = /etc/postfix
daemon_directory = /usr/libexec/postfix
data_directory = /var/lib/postfix
debug_peer_level = 2
debugger_command = PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin ddd $daemon_directory/$process_name $process_id & sleep 5
html_directory = no
inet_interfaces = all
inet_protocols = ipv4
mail_owner = postfix
mailq_path = /usr/bin/mailq.postfix
manpage_directory = /usr/share/man
mydestination = $myhostname, localhost.$mydomain, localhost
newaliases_path = /usr/bin/newaliases.postfix
queue_directory = /var/spool/postfix
readme_directory = /usr/share/doc/postfix-2.10.1/README_FILES
relayhost = [smtp.gmail.com]:587
sample_directory = /usr/share/doc/postfix-2.10.1/samples
sendmail_path = /usr/sbin/sendmail.postfix
setgid_group = postdrop
smtp_always_send_ehlo = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_tls_security_level = verify
unknown_local_recipient_reject_code = 550
```

#### 8 Set Sender Email Address in /etc/aliases File

```bash
[root@nagios ~]# vim /etc/aliases
root:       Mail_ID@gmail.com
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



