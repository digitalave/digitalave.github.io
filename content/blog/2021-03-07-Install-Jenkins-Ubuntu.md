---
title: 'How To Setup Jenkins On Ubuntu 20.04LTS'
subtitle: "Automate Application Build and Deployments Using Jennkins CI/CD Pipelines"
description: "This tutorial walk you through how to setup Jenkins on Ubuntu 20.04LTS and Debian. Jenkins is an open-source continues integration and continues deployment tool. Which can use to automate application building, testing and deploying. Jenkins is the most popular automation server.  Jenkins built using java, which can integrate with numerous plugins."
image: /img/post-imgs/Jenins-Install-ubuntu/Jenkins-Ubuntu.jpg
categories: ["DevOps"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2021-03-07
---

# Setup Jenkins On Ubuntu 20.04LTS

Jenkins is an open-source continues integration and continues deployment tool. Which can use to automate application building, testing and deploying. Jenkins is the most popular automation server.  Jenkins built using java, which can integrate with numerous plugins.

## Prerequisites

OS Requirement: Make sure to use Ubuntu LTS versions

Hardware Requirement:

| Minimum Hardware Requirement | Recommended Hardware Requirement |
|------------------------------|---------------------------------|
| 1GB+ Hard Disk Space         | 50GB+ Hard  Disk Space          |
| 256 MB RAM                   | 1GB + RAM                       |

Software Requirement:

*  Java - JRE8/11 (32Bit or 64Bit Supported)

> Note: Older versions and Java 9, 10,12 are not supported

| JDK                     | JRE    |
|-------------------------|--------|
| OpenJDK 8 , OpenJRE 8   | JRE 8  |
| OpenJDK 11 , OpenJRE 11 | JRE 11 |
|                         |        |

### STEP 01: Update OS

```bash
sudo apt update -y
```

### STEP 02: Install Java

Jenkins is built with Java, So we need to install the appropriate Java version. This time I'm going to use OpenJDK version 11.
Now head-over to your ubuntu terminal and do the following steps.

Run this command and pick one OpenJDK version 8 or 11 from the list

```bash
sudo apt search openjdk
```

```bash
sudo apt-get install openjdk-11-jdk -y
```

```bash
dimuthu@build-svr:~$ java -version
openjdk version "11.0.10" 2021-01-19
OpenJDK Runtime Environment (build 11.0.10+9-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.10+9-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)
```

### STEP 03: Add GPG Key and Repository

Install GPG Trusted Key 
```bash
sudo wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```
Add Jenkins Repository

This step is to append the Jenkins repository in the Debian source.list

```bash
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    
```

Once again, update OS repositories.

```bash
sudo apt-get update -y
```

Install Jenkins


```bash
sudo apt-get install jenkins -y
```

### STEP 04: Enable & Start Jenkins

Start & enable Jenkins service on system boot.

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Optionally, Sometimes you need to allow TCP port 8080 though out the firewall. If you are using Ubuntu, execute the following command.

```bash
sudo ufw allow 8080
```

### STEP 05: Access Jenkins

Now, You can access your Jenkins server through the web browser.

`http://<YOUR-HOST-NAME-OR-IP>:8080/`

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/1.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

For the 1st time, Jenkins will prompt to enter an unlock password. You need to execute the following command on your terminal and copy the output password and paste it into the Administrator password text box.

`Troubleshooting: If the "initialAdminPassword" not available, You may have to remove jenkins and try again.`

**REF:** <a href="https://stackoverflow.com/questions/48611411/initialadminpassword-file-is-not-created-in-jenkins-folder-in-windows-10-os" target="_blank">Admin Password Not Available</a>

For the first time, Jenkins will prompt us to unlock, and it will tell us to copy the password from this location. Copy this 32charactor alphanumeric password and paste it into the text box in the wizard.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

`204bedca295343e483335ae2f26e75f4`

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/2.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

### STEP 07: Install Plugin

In this section, You will need to install Jenkins plugins according to how going to use them. You can choose either option. 

In this case, I'm choosing the "select plugins to install option."

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/3.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Once you choose the required plugins, click next.
This plugin installation may take considerable time. Please wait until completed.

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/4.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/5.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

### STEP 08: Create First Admin User

Next, You have to provide your "name", username", "password", and "email" for the Jenkins admin user.

Next, You need to provide your FQDN or IP address. By default, the IP address will automatically load into the "instance URL" section.

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/7.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/8.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/Jenins-Install-ubuntu/9.png" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

Now, Jenkins installation has been completed successfully.
This is the 1st session of the Jenkins tutorial series. If you love to learn more about Jenkins, refer to my other articles available on this website.

If you are facing issues with the installation, please comment below. I will regularly reply here. 