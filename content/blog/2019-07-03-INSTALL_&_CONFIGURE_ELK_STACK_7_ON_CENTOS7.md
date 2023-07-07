---
title: 'Install and Configure ELK Stack 6.8 On CentOS7'
image: /img/post-imgs/elk-stack/ELK.png
categories: ["DevOps"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2019-07-03
---

{{< youtube 2QofuOkobKc >}}

## INTRODUCTION

“ELK” is the acronym for the three open source projects call Elasticsearch, Logstash and Kibana. ELK stack made easier to analyze logs to system administrators. ELK stack collect logs from clients using Beats protocol

## ELK STACK MAIN COMPONENTS

**Elasticsearch** is an open source, distributed, RESTful, JSON based search and analytic engine. Easy to use and flexible. Elasticsearch is the heart of ELK stack. Elasticsearch is a No-SQL database.

**Logstash** is a open source, server-side data processing pipeline that pull events data from multitude of sources simultaneously, transform it, and then sends it to Elasticsearch. Easily pull data from logs, metrics, web applications, data sources and various AWS services. Logstash dynamically transforms and prepares data regardless of format or complexity. Derive structure from unstructured data with grock.

**Kibana** provides GUI for users visualize data with charts and graphs real-time. It is a window into the Elastic Stack. Provides data exploration, visualization and dashboarding.

**Beats** is the platform for single-purpose data shippers. They install as lightweight agents and send data from numerous machines to Logstash of 

{{< image src="/img/post-imgs/elk-stack/image002.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/elk-stack/image003.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Software Versions I have used in this tutorial.

ELK Stack Server 192.168.10.10 (CentOS7)
Beat Client 192.168.0.49 (CentOS7)
Elasticsearch Version: 6.8 
Logstash Version 6.8 
Kibana Version 6.8

## STEP 1: COMPLETE PREREQUSITES

#### SET HOSTNAME

```bash
vim /etc/hostname
```


```bash
vim /etc/hosts
```

#### CLEAR AND REMOVE YUM CACHE

```bash
sudo rm /etc/yum.repos.d/$REPONAME.repo
yum clean all
```

Delete the yum cache for the repo

```bash
sudo rm -rf /var/cache/yum/x86_64/6/$REPONAME
```


Clearing the yum Caches

```bash
su -c 'yum clean headers'
su -c 'yum clean packages'
su -c 'yum clean metadata'
```


### Install and Update Latest RPM Repositories

```bash
rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

rpm -ivh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

rpm -ivh https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm

rpm -ivh http://repository.it4i.cz/mirrors/repoforge/redhat/el7/en/x86_64/rpmforge/RPMS/rpmforge-release-0.5.3-1.el7.rf.x86_64.rpm
```

## STEP 2: INSTALL JAVA JDK

Java is required for the Elastic stack deployment. Elasticsearch requires Java 8, it is recommended to use the Oracle JDK 1.8. I will install Java 8 from the official Oracle rpm package. ELK requires the Oracle Java JDK package has to be installed. The same JVM version should be installed on all Elasticsearch nodes and clients.

### INSTALL JDK RPM

```bash
rpm -Uvh jdk-8u211-linux-x64.rpm
```


### SET DEFAULT JAVA VERSION

```bash
alternatives --config java

alternatives --set jar /usr/java/jdk1.8.0_211-amd64/bin/jar

alternatives --set javac /usr/java/jdk1.8.0_211-amd64/bin/javac
```


### SET JAVA ENVIRONMENT VARIABLES
SET JAVAC AND JAR PATHS

```bash
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64/
export JRE_HOME=/usr/java/jdk1.8.0_211-amd64/jre/
export PATH=$PATH:/usr/java/jdk1.8.0_211-amd64/bin/:/usr/java/jdk1.8.0_211-amd64/jre/bin/
```

`vim ~/.bashrc`


```bash
export JAVA_HOME=/usr/java/jdk1.8.0_211-amd64/
export JRE_HOME=/usr/java/jdk1.8.0_211-amd64/jre/
export PATH=$PATH:/usr/java/jdk1.8.0_211-amd64/bin/:/usr/java/jdk1.8.0_211-amd64/jre/bin/
```


### CHECK JAVA VERSION
```bash
java -version
```


## STEP 3: INSTALL AND CONFIGURE ELASTICSEARCH

In this step, I will install and configure Elasticsearch version 6.8

#### IMPORT PUBLIC GPG KEY TO THE ELK-STACK SERVER

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```


#### CREATE YUM REPO FILE FOR ELASTICSEARCH

`vim /etc/yum.repos.d/elasticsearch.repo`

```bash
[logstash-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```


#### INSTALL ELASTICSEARCH YUM PACKAGES

```bash
sudo yum -y install elasticsearch
```


#### CONFIGURE ELASTICSEARCH

Do the following changes

`vim /etc/elasticsearch/elasticsearch.yml`

```bash
cluster.name: elk
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 192.168.10.10
http.port: 9200
```

{{< image src="/img/post-imgs/elk-stack68/elastic1.PNG" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

`vim /etc/elasticsearch/jvm.options`

```bash
-Xms4g
-Xmx4g
```


`FIREWALL CONFIGURATION`

Allow traffic through the TCP port 9200 in the firewall.

```bash
firewall-cmd --permanent --add-port=9200/tcp
firewall-cmd --permanent --add-port=9300/tcp
firewall-cmd --reload
```


#### START & ENABLE ELASTICSEARCH AT SYSTEM BOOT

```bash
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl restart elasticsearch.service
systemctl status -l elasticsearch.service
```

#### TEST ELASTICSEARCH

Check Elasticsearch port “9200” state as “LISTEN”

```bash
netstat -plntu
```


#### OPEN IN BROWSER 

```bash
http://192.168.10.10:9200/?pretty
```


#### OPEN IN TERMINAL

```bash
curl -XGET '192.168.10.10:9200/?pretty'
```

## STEP 4: INSTALL AND CONFIGURE LOGSTASH

In this step I will install Logstash version 6.8 and configure it as a central log server, receives logs from clients with Filebeat, then filter and transform the syslog data and move it into the stash (Elasticsearch)

#### IMPORT PUBLIC GPG KEY TO THE ELK-STACK SERVER

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

#### CREATE YUM REPO FILE FOR ELASTICSEARCH

`vim /etc/yum.repos.d/logstash.repo`

```bash
[logstash-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```


#### INSTALL LOGSTASH YUM PACKAGES

sudo yum -y install logstash

NOTE: Need to genarate SSL Certificate if you using SSL.

GENERATE A NEW SSL CERTIFICATE
Create new ssl certificate for securing communication between Logstash & Filebeat (clients). SSL Certificate file use clients to identify the elastic server.

Do the following changes under the “[ V3_ca ]” section for the server identification.

`vim /etc/pki/tls/openssl.cnf` 

```bash
[ v3_ca ]
#Server IP Address
subjectAltName = IP: 192.168.10.10
```


Generate the certificate file with the openssl command.

```bash
openssl req -config /etc/pki/tls/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout /etc/pki/tls/private/logstash-forwarder.key -out /etc/pki/tls/certs/logstash-forwarder.crt
```


Once ssl certificate is ready, this certificate should be copied to all the clients using scp command.

### CONFIGURE LOGSTASH

`vim /etc/logstash/logstash.yml`

```bash
path.data: /var/lib/logstash
http.host: "192.168.10.10"
path.logs: /var/log/logstash
```

{{< image src="/img/post-imgs/elk-stack68/logstash1.PNG" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

#### JVM configuration

`vim /etc/logstash/jvm.options`

```bash
-Xms2g
-Xmx2g
```

#### Create Following Files Under /etc/logstash/conf.d/ Directory.

`vim /etc/logstash/conf.d/auditbeat.conf`

```bash
### INPUT SECTION ###
### This section make Logstash to listen on port 5044 for incoming logs & provides SSL certificate for secure connection.
input {
  beats {
    port => 5044
#   ssl => true
#   ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
#   ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}

### FILTER SECTION ###
### This section parse the logs before sending them to Elasticsearch.
filter {
if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGLINE}" }
    }
    date {
match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
}
}
}

### OUTPUT SECTION ###
### This section defines the storage for the logs to be stored.
output {
  elasticsearch {
    hosts => ["http://192.168.10.10:9200"]
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.DD}"
    document_type => "%{[@metadata][type]}"
  }
}
```

#### FIREWALL CONFIGURATION
Allow traffic through the TCP port 5044 in the firewall.

```bash
firewall-cmd --permanent --add-port=5044/tcp
firewall-cmd --reload
```


#### ENABLE & START LOGSTASH SERVICE

```bash
systemctl enable logstash.service 
systemctl restart logstash.service 
systemctl status -l logstash.service
```

## STEP 5: INSTALL AND CONFIGURE KIBANA

#### IMPORT PUBLIC GPG KEY TO THE ELK-STACK SERVER

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```


#### CREATE YUM REPO FILE FOR KIBANA

`vim /etc/yum.repos.d/kibana.repo` 

```bash
[kibana-6.x]
name=Kibana repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```


#### INSTALL KIBANA YUM PACKAGES

```bash
sudo yum install kibana
```


#### CONFIGURE KIBANA

`vim /etc/kibana/kibana.yml`
```bash
server.port: 5601
server.host: "0.0.0.0"
server.name: "elk"
elasticsearch.hosts: ["http://192.168.10.10:9200"]
```

{{< image src="/img/post-imgs/elk-stack68/kibana1.PNG" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

#### FIREWALL CONFIGURATION
Allow traffic through the TCP port 5044 in the firewall.

```bash
firewall-cmd --permanent --add-port=5601/tcp
firewall-cmd --reload
```


#### ENABLE & START LOGSTASH SERVICE

```bash
systemctl daemon-reload 
systemctl enable kibana.service
systemctl restart kibana.service
```


## STEP 6: INSTALL AND CONFIGURE NGINX

#### INSATLL EPEL REPOSITORY

```bash
yum install epel-release
```


#### INSTALL NGINX & HTTPD-TOOLS

```bash
yum install nginx httpd-tools
```


CREATE USERNAME “ADMIN” AND PASSWORD “123456” FOR KIBANA WEB INTERFACE

```bash
htpasswd -c /etc/nginx/htpasswd.kibana admin
```


#### CONFIGURE NGINX
Edit the Nginx configuration file and remove the ‘server { }’ block, so we can add a new virtual host configuration.

`vim /etc/nginx/nginx.conf`

COMMENT {Server} Block:

Create new virtual host configuration file named “kibana.conf” under the conf.d directory.

#### Create VHOST FOr KIBANA:

`vim /etc/nginx/conf.d/kibana.conf` 

```bash
server {
    listen 80;

    server_name elk-stack.co;

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.kibana;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
   }
```

{{< image src="/img/post-imgs/elk-stack68/nginx_kibana.PNG" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

#### Check Nginx Configuration

```bash
nginx -t
```


#### FIREWALL CONFIGURATION
Allow traffic through the TCP port 80 in the firewall.

```bash
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```


#### ENABLE & START NGINX SERVICE

```bash
systemctl enable nginx.service 
systemctl restart nginx.service
```


#### SELINUX CONFGURATION

```bash
setsebool -P httpd_can_network_connect 1
```

## STEP 07: Connect Kibana Frontend With Elasticsearch

##### You need assign Kibana to which Elasticsearch indeces you want yo explore.

Configure the Elasticsearch Indices what you want to access with Kibana.

Open Web Browser and Point To...
(Only via Kibana)
`http://YOURIP.com:5601`

OR

(If nginx/apache proxy redirect with VHOST)
`http://YOURIP.com:80`

{{< image src="/img/post-imgs/elk-stack68/1_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Navigate To

`Managemnt >> Kibana >> Create Index Pattern`

{{< image src="/img/post-imgs/elk-stack68/6_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Now Navigate To 

`Discover >> (Now You Search For Logs By Available Fields)`

{{< image src="/img/post-imgs/elk-stack68/3_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

### Bottom Line: 

##### Hope you guys get some idea about how to install ELK Stack on CentOS7 step by step. And monitor system logs in a effective manner.

## In the next lesson I will teach you following points.
1. How to Install Beat Log Collectors on Client Systems.
2. How to Search Log Data and Narrow down them into your Requirement
3. How To Save Search Data
4. How To Visualize Data
5. How To Create Dashboards 
6. How To Use Dev Tools 
7. Log Rotate Effectively
8. Optimize Logstash (Increase Index Search Performance & Lower Hardware Requirements) 

**Video Tutorial on YouTube Will Be Available Soon.**

{{< image src="/img/post-imgs/elk-stack68/5_N.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
