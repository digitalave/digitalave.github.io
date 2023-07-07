---
title: 'How to Install and Configure Jenkins on Ubuntu 18.04 LTS 16.04 Debian'
image: /img/post-imgs/jenkins-ubuntu/Jenkins.jpg
categories: ["DevOps"]
date: 2020-04-06
---

{{< youtube rqJPEACdrmM >}}

### Introduction:

Jenkins is an open-source automation server that provides hundreds of plugin to perform continuous integration and continues delivery to build and deploy projects.

Continuous integration (CI) is a DevOps practice in which team members regularly commit their code changes to the version control repository. Automated builds and tests are run. Continuous delivery (CD) is a series of practices where code changes are automatically built, tested and deployed to production.

If you are a software developer, surely Jenkins suites and automates your CI/CD build tasks quickly.

Jenkins can automate continuous integration and continuous delivery (CI/CD) for any project.
Support for hundreds of plugins in the update centre.  Which provides infinite possibilities for what Jenkins can do.
Jenkins can configure into a distributed system that distributes work across multiple node/machines.

### Features: 

**CICD** - Continues Integration and Continues Delivery

**Plugins** - Hundreds of Plugin Support

**Extensible** - Extended possibilities using its Plugins

**Distributed** - Distribute work across multiple machines and projects

### Prerequisites:

`Hardware - RAM > 1GB | HDD > 50GB+`
`Software - JAVA (JRE 8 | JDK 11)`

Following JDK/JRE Versions support for current Jenkins versions

`OpenJDK JDK / JRE 8 - 64 bits`
`OpenJDK JDK / JRE 11 - 64 bits`

**NOTE: Always check JAVA version requirement before proceeding with the Jenkins installation.**

REF: <a href="https://www.jenkins.io/doc/administration/requirements/java/" target="_blank">https://www.jenkins.io/doc/administration/requirements/java/</a>

### STEP 01: Install Oracel/Open JDK 11

Now, I'm going to install OpenJDK 11 on my system.

##### Install OpenJDK

```bash
sudo apt update

sudo apt-get install openjdk-11-jdk -y
```
OR

REF: <a href="https://www.oracle.com/java/technologies/javase-jdk11-downloads.html" target="_blank">https://www.oracle.com/java/technologies/javase-jdk11-downloads.html</a>

```bash
sudo dpkg -i jdk-11.0.7_linux-x64_bin.deb
```

##### Set Default JDK Version

```bash
sudo update-alternatives --config java

java -version
```

### STEP 02: Install Jenkins Using Debian Repository

##### Import trusted PGP Key for Jenkins

```bash
sudo wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
```

##### Add Jenkins Repository

```bash
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
```

```bash
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

##### Install Jenkins

```bash
sudo apt-get update

sudo apt-get install jenkins
```

Jenkins service will automatically start after the installation process is complete. You can verify it by printing the service status.

```bash
sudo systemctl status jenkins
```

If it isn't enabled,/started automatically.

```bash
sudo systemctl enable jenkins

sudo systemctl start jenkins
```

### STEP 03: Configure Firewall

Allow port 8080 through the Ubuntu firewall.

```bash
sudo ufw allow 8080/tcp
```

### STEP 04: Initial Setup For Jenkins

Once the above configuration completed, Open-up your web browser and access it through the IP: PORT.

> http://your_ip_or_domain:8080 

Now, Head-over to the terminal again, and find out the Administrator password using this command.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

{{< image src="/img/post-imgs/jenkins-ubuntu/1.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Copy the password from the terminal and paste it into the required field



The next screen at the initial setup wizard will ask for Install suggested plugins or select specific plugins. Click on the Install suggested plugins box, and the installation process will start immediately.

{{< image src="/img/post-imgs/jenkins-ubuntu/2.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/jenkins-ubuntu/3.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Once the plugins are installed, you will be prompted to set up the first admin user. Fill out all required information and click Save and Continue.

{{< image src="/img/post-imgs/jenkins-ubuntu/4.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/jenkins-ubuntu/5.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Click on the Start using Jenkins button. You will be redirected to the Jenkins dashboard logged in as the admin user you have created in one of the previous steps.

{{< image src="/img/post-imgs/jenkins-ubuntu/6.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}
{{< image src="/img/post-imgs/jenkins-ubuntu/7.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Now you've successfully installed Jenkins on your Ubuntu system.

######  Bottom Line

###### I hope you learned how to install Jenkins on Ubuntu. And In the next tutorial, I'll show you how to integrate GitLab with your newly built Jenkins server.

###### If you have any questions, please leave a comment on the YouTube comment section. 









