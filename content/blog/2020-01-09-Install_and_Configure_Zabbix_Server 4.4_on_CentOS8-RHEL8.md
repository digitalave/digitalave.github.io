---
title: 'Install and Configure Zabbix 4.4 on CentOS8 RHEL8'
image: /img/post-imgs/Zabbix4.4/zabbix_44.jpg
categories: ["Monitoring"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2020-01-09
---

{{< youtube aCCBNLOYO94 >}}

**In this tutorial, I'm going to cover up how to install and configure Zabbix 4.4 Monitoring Server on CentOS8**

### Introduction:

Zabbix is an enterprise-class free open source monitoring solution.

Newly released Zabbix Monitoring solution empowered with unlimited capabilities of monitoring any IT device in house.

Which allows you to keep eye on any kinds of IT infrastructure, services, applications and resources.

You will get ability to monitor any performance metrics and incidents in your network such as Power status, System Status, CPU, Memory and Network bandwidth usage, Packet Loss rate, Link status, New device added or removed. And many more...

* Which provides real-time graphing and highly configurable graphing facility to users.
* Available thousands of templates on  the Zabbix share. Which is a community to share thousands of templates and scripts accomplish the monitoring tasks.
* Automatic network discovery provides agent auto registration facility.
* Zabbix API provides interface for 3rd party software integration such as Grafana and many more.

For this tutorial, I will  going to install Zabbix using yum destitution package for CentOS and RHEL.

### STEP 01: Configure Prerequisitess

##### A. Set Resolvable Hostname:

Now, I'm going to set host-name with fully qualified domain name(FQDN). 

```bash
vim /etc/hostname
```

```bash
vim /etc/hosts
```

##### B. Install Latest EPEL Release:

Now, I'm going to install latest epel release 

```bash
rpm -ivh https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/e/epel-release-8-7.el8.noarch.rpm
```

##### C. Update Operating System:

Then, Update the Operating System.

```bash
yum -y update
```

##### D. Configure SELinux:

You need to execute these commands, If you have enabled SELinux in "enforcing" mode.

```bash
setsebool -P httpd_can_connect_zabbix on

setsebool -P httpd_can_network_connect_db on
```

Either, You also can set SELinux into "**permissive**" mode 

```bash
vim /etc/selinux/config 
SELINUX=permissive
```

##### E. Reboot the System:

### STEP 02: Install Apache HTTPD Web Server

##### A. Install HTTPD Package:

Now, I'll going to install httpd package for web server.

```bash
yum install httpd httpd-tools
```

##### B. Enable & Start HTTPD Service:

Enable service to start automatically at the system boots.

```bash
systemctl enable httpd.service
```

Then,  Start service.

```bash
systemctl start httpd.service
```
##### C. Allow HTTPD Through The Firewall:

Allow port 80 for Apache service through out the system firewall.

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload
```

##### D. verify The Service is Running:

Finally, Check weather the Apache service and listen through the port 80

```bash
netstat -tulpen | grep httpd
```

Now, goto /var/www/html/ directory and create a index.html file.

```bash
vim /var/www/html/index.html
```


`"THIS IS A TEST PAGE"`

Save and exit.

Then, Open up the web browser and hit the server's IP address

`http://<your_server_ip_address>/`


OK, Now you'll see Apache testing page. Now Apache configuration has been completed.

Let's head-over to next step.

### STEP 03: Install & Configure PHP

##### A. Install PHP & Dependencies: 

Now, I'm going to install PHP and required PHP dependencies which required to run Zabbix fronted.

I Here, PHP used to gather matrices from MariaDB/Mysql database and process to display dynamic content via Apache web server

```bash
yum -y install php php-cli php-common php-devel php-pear php-gd php-mbstring php-mysqlnd php-xml php-bcmath
```

Then, We need to configure "**php.ini**" file with these variables.

##### B. Configure PHP:

```bash
vim /etc/php.ini
```

OR

```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

```bash
max_execution_time = 600
max_input_time = 600
memory_limit = 1024M
post_max_size = 32M
upload_max_filesize = 32M
date.timezone = Asia/Colombo
```

##### C. Test PHP:

Finally, Create a "**info.php**" file to check whether PHP working well.

```bash
vim /var/www/html/info.php
<?php phpinfo(); ?>
```

Restart Apache service again.

```bash
systemctl restart httpd.service
```

Then, Open the web browser and hit server's IP address with "info.php". The you would see a page like this.

`http://<your_server_ip_address>/info.php`

{{< image src="/img/post-imgs/Zabbix4.4/zabbix-1.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

### STEP 04: Install and Configure MariaDB

##### A. Install MariaDB:

Then, Install MariaDB on our Zabbix hosting server.

```bash
yum install mariadb mariadb-server
```

##### C. Allow MySQL Service Through The Firewall:

Allow MariaDB/MySQL service through the firewall.

```bash
firewall-cmd --permanent --add-service=mysql 
firewall-cmd --reload
```

Start and enable Mariadb/MySQL service on reboot.

```bash
systemctl start mariadb.service
systemctl enable mariadb.service
```

##### D. MySQL Initial Setup:

MariaDB Initial configuration:

```sql
mysql_secure_installation

Set root password? [Y/n] Y

New password:

Re-enter new password:

Remove anonymous users? [Y/n] Y

Disallow root login remotely? [Y/n] Y

Reload privilege tables now? [Y/n] Y
```

Log into MariaDB Server:

```bash
mysql -uroot -p

MariaDB [(none)]> select version();
```

```bash
mysql -V
```

### STEP 05: Create "zabbixdb" database and "zabbixuser" user

##### A. Create a Zabbix User & Dsatabase: 

Now, I'm going to create a Zabbix user and a database.
In this time, I'm going use "**zabbixuser**" as a user and "**zabbixdb**" as a database.

First login into MariaDB/MySQL.

```bash
mysql -uroot -p
Enter password:
```

Then create a database.

```sql
CREATE DATABASE zabbixdb CHARACTER SET UTF8 COLLATE UTF8_BIN;
```

Grant privileges to zabbixuser and zabbixdb with a password.

```sql
GRANT ALL PRIVILEGES ON zabbixdb.* to zabbixuser@'localhost' IDENTIFIED BY 'PASSWORD';
```

```sql
GRANT ALL PRIVILEGES ON zabbixdb.* to zabbixuser@'%' IDENTIFIED BY 'PASSWORD';
```

Refresh permissons.

```sql
FLUSH PRIVILEGES;

QUIT;
```

Now, I have completed the installation of LAMP stack which include Apache, PHP and MariaDB.

Now we can go straight forward to the Zabbix package installation and configuration.

### STEP 06: Install & Configure Zabbix Server 

First,  We need to import PGP trusted signing key.

```bash
rpm --import http://repo.zabbix.com/RPM-GPG-KEY-ZABBIX
```

Next, Install the repository configuration package for the Zabbix.

**RHEL8 / CENTOS8**

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/4.4/rhel/8/x86_64/zabbix-release-4.4-1.el8.noarch.rpm
```

**RHEL7 / CENTOS7**

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/4.4/rhel/7/x86_64/zabbix-release-4.4-1.el7.noarch.rpm
```

Then, Install Zabbix Server.

```bash

yum install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf
```

{{< image src="/img/post-imgs/Zabbix4.4/zabbix-2.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}


Finally, Import Zabbix server database schema.

```bash
zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -uzabbixuser -pPASSWORD zabbixdb
```

`NOTE: This may take a while to import the database schema. So, Please wait untill it done.`

Configure Zabbix Server:

Let's headover to Zabbix server main configuration file which located in "**/etc/zabbix/zabbix_server.conf**" and do these changes.

```bash
vim /etc/zabbix/zabbix_server.conf
```

In here you need to Zabbix server's IP address, MySQL or MariaDB username and password

```bash
## DATABASE NAME 
DBName=zabbixdb

## DATABASE USERNAME
DBUser=zabbixuser

## DATABASE PASSWORD
DBPassword=PASSWORD
```

Allow Zabbix Service Through The Firewall:

```bash
firewall-cmd --permanent --add-port=10050/tcp
firewall-cmd --permanent --add-port=10051/tcp
firewall-cmd --reload
```

Set Zabbix server name:

```bash
vim /etc/httpd/conf/httpd.conf
```

```nginx
ServerName zabbix.da.com:80

ServerAdmin zabbix@da.com
```

```nginx
systemctl restart httpd.service
```

**Configure PHP for Zabbix Frontend:**

Let's move to Zabbix server's Apache configuration file.

```bash
vim /etc/httpd/conf.d/zabbix.conf
```

Some PHP values has be configured. These values you may be need to change later when your requirements getting higher. It depends on your Zabbix server where you going to use. Don't forget to uncomment and set your Time Zone.

```bash
vim /etc/httpd/conf.d/zabbix.conf
```

```bash
Allow from All
```

```bash
php_value max_execution_time 600
php_value memory_limit 1024M
php_value post_max_size 32M
php_value upload_max_filesize 32M
php_value max_input_time 600
php_value max_input_vars 10000
php_value always_populate_raw_post_data -1
php_value date.timezone Asia/Colombo
```

Start & enable service to start at system boot.

```bash
sudo systemctl enable zabbix-server 
sudo systemctl restart zabbix-server
```

### STEP 07: Further SELinux Configuration For Zabbix

If your SELinux configuration is still on "**enforcing**" mode. then you need to execute these commands to make connection between Zabbix frontend and the server.

```bash
setsebool -P httpd_can_connect_zabbix on
```

### STEP 08: Restart & Enable All Services Again.

Now, You I'll enable & restart all services once.

```bash
systemctl enable httpd.service mariadb.service zabbix-server.service
systemctl restart -l  httpd.service mariadb.service zabbix-server.service
```

And Check whether they are running well.

```bash
systemctl status -l  httpd.service mariadb.service zabbix-server.service
```

Now, We are almost ready to complete the Zabbix server setup.
Let's Head over to the web browser and enter Zabbix server's IP address with the "**zabbix**" suffix. and hit enter.

`http://zabbix_server_IP/zabbix/`

For the first time you may need to log into  the system using **default** username and password.

Default Zabbix Username: **Admin**

Default Zabbix Password: **zabbix**

Then, follow the instruction as seen on installation wizard.

{{< image src="/img/post-imgs/Zabbix4.4/zabbix-3.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/Zabbix4.4/zabbix-4.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/Zabbix4.4/zabbix-5.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/Zabbix4.4/zabbix-6.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/Zabbix4.4/zabbix-7.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/Zabbix4.4/zabbix-8.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/Zabbix4.4/zabbix-9.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/Zabbix4.4/zabbix-10.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}