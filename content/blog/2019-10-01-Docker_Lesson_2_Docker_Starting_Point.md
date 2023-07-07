---
title: 'Lession 02 - Docker Basics - Where Am I Start Learn Docker'
image: "/img/post-imgs/Docker-Installation/docker_N.jpg"
categories: ["devops"]
comments: true
type: "regular" # available types: [featured/regular]
draft: false
date: 2019-10-01
---

In this tutorial I'm going to teach you the basics you should have known before dive into depth.

Now, I'm going to cover up fundamentals you must know.

#### Understand Docker Images:

A. List Docker Images

$docker images

Repository - Image where it came from 
TAG - Version Number
Image ID - Internal representation of Docker Image

B. Running an Image

$docker run -ti ubuntu:latest bash 

Docker image tuns into a running container. Docker run takes images into container.

-ti  Terminal Interactive - stands for interactive shell (Which help  for  TAB Completion & work correctly)

bash - Run command on a bash shell

Now I'm in a Ubuntu  environment

Now check 

#cat /etc/lsb-release 

To exit

#exit


D. List the Running Images

$docker ps

ID : Running Container's ID
IMAGE : Image which made from
COMMAND : Command it running on
STATUS  
PORT


E. List the Recently Stopped Last Containers 

$docker ps -l

F. List All Stopped Containers

$docker ps -al

G. Docker Commit 
Docker Commit takes containers back to new images.

$docker run -ti ubuntu:latest bash 

#touch test.file

#exit

$docker ps -l

ID : XXXXXXXX
IMAGE : ubuntu
COMMAND  : bash 
CREATED :
STATUS : 
PORTS :
NAMES : ubuntu_happy

$docker commit <Container ID>

OR

$docker commit <NAMES>

Which returns Image ID
sha256:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

Now, I have a newly created Docker Images.

H. Give Images a Name Using TAG

$docker tag <Image ID> my-new-image

Now List Docker Images

$docker images

I. Now, Run Docker Image using The TAG

$docker run -ti my-new-image bash

#ls -la /test.file

Now, It has my  test.file in it.

###################################

How To Run Processes In Docker Containers

$docker run --rm -ti ubuntu sleep 5

$docker run -ti ubuntu bash -c "sleep 3; echo  all done"

--rm Delete this container when it exits 

-ti Interactive Terminal

Keep Things Running  on Container

$docker run -d -ti ubuntu

-d detach things running and leaving  running in the background 

Now, Jump into  the task again that I had started over there.

$docker attach <container_name>

By Pressing  Ctrl+P, Ctrl+Q these key combinations you can exit from  terminal by detaching from it. But, leaves it running. So you can attach back again. 

again you can find running  docker container using "docker ps" and you can run it again using "docker attach <container name>"

Docker Logs:
By looking at the container output. Even though, you start the container, but it didn't work. 
Now "docker logs" command come to  an action.

$docker run --name example -d ubuntu bash -c "lose /etc/passwd"
docker logs <container_name> use look at what the output was.

docker logs <container_name>

$docker logs exmaple

Stopping  & Removing Containers:


docker kill <container_name> Its stop the container.

docker rm <container_name> Its remove the container.

$docker kill <container_name>
docker rm <container_name>
$docker ps -l


Docker Resource Limit Enforce Management:
Docker has a feature to limit & enforce limit the resources. 
You can limit it to  a fixed amount of memory as you need.

#### Docker Commands

##### Docker Management Commands

1. Docker "help" Command:

Using "help" command you can see what are the other available commands. 

```bash
docker -h | more
```

2. To all the management command which associated with "docker image" command.


```bash
[root@svr1 ~]# docker image --help

Usage:	docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
```



3. List Docker Images:

```bash
docker image ls
```

{{< image src="/img/post-imgs/Docker-Installation/2.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

4. Pull Docker image: /assets/img/post-imgs/

Pull an image or a repository from a registry (Docker Hub).

```bash
docker image pull nginx
```

{{< image src="/img/post-imgs/Docker-Installation/3.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}


5. View Image Information:

```bash
docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              f949e7d76d63        5 days ago          126MB
httpd               latest              19459a872194        2 weeks ago         154MB
hello-world         latest              fce289e99eb9        9 months ago        1.84kB

docker image inspect fce289e99eb9
```

{{< image src="/img/post-imgs/Docker-Installation/4.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}



6. To list the all all command related to "docker container" command.

```bash
[root@svr1 ~]# docker container --help

Usage:	docker container COMMAND

Manage containers

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  inspect     Display detailed information on one or more containers
  kill        Kill one or more running containers
  logs        Fetch the logs of a container
  ls          List containers
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  prune       Remove all stopped containers
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  run         Run a command in a new container
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker container COMMAND --help' for more information on a command.
```


7. List Running Containers:


```bash
docker container ls
```

{{< image src="/img/post-imgs/Docker-Installation/5.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

8. Run Command in a new container:

```bash
docker container  run nginx:latest
```

8. List All Containers:

```bash
docker container ls -a
```

{{< image src="/img/post-imgs/Docker-Installation/6.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

9. Run Docker Container:

```bash
docker container run  -P -d nginx
```

-P Port

-d Run program in the background

{{< image src="/img/post-imgs/Docker-Installation/7.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}


```bash
[admin@svr1 ~]$ curl http://172.17.0.3
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

10. List all the running Containers

```bash
docker container ps
```

{{< image src="/img/post-imgs/Docker-Installation/8.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

11. Display detailed information on the specific container:


```bash
docker container inspect 4d625f4a7921
```

```bash
docker container inspect <CONTAINER_ID>
```

{{< image src="/img/post-imgs/Docker-Installation/9.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

12. Display the running process of specific container:


```bash
docker container top 4d625f4a7921
```

```bash
docker container top <CONTAINER_ID>
```

{{< image src="/img/post-imgs/Docker-Installation/10.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

13. Attach Docker Containers:

```bash
docker container run -P -d nginx:latest 

docker container ls -a
```




{{< image src="/img/post-imgs/Docker-Installation/12.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

{{< image src="/img/post-imgs/Docker-Installation/11.jpg" class="img-fluid" title="Digital Avenue DevOps Tutorials" webp="true" >}}

```bash
[root@svr1 ~]# docker container attach 7e9428fa3cbc
172.17.0.1 - - [01/Oct/2019:09:07:57 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```
14. View Stats for Specific Docker Container

```bash
docker container stats 7e9428fa3cbc
```


15. Run Command in a Specific Container:
Which can use for execute commands on the container.

```bash
[root@svr1 ~]# docker container exec -it 7e9428fa3cbc /bin/bash
root@7e9428fa3cbc:/# ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
```
16. Remove Docker Container:

```bash
docker container rm -f 7e9428fa3cbc

docker container rm -f <CONTAINER_ID>
```

17. Remove all the Stopped Containers:

```bash
docker container prune
```
###############################################################################


##### Creating Docker Containers

-d Detach from the container run the container in the background.

```bash
[root@svr1 ~]docker container run -d nginx
8317663311f0d2d987ccae41d8c85802ef5ed93512f5f79479901ed27857bad9
[root@svr1 ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
8317663311f0        nginx               "nginx -g 'daemon of…"   8 seconds ago       Up 7 seconds        80/tcp              hardcore_leakey
```

Here it can see running in the background.

-i Keep "STDIN" open

```bash
[root@svr1 ~]# docker container run -it ubuntu:latest 
root@fdc372caedf7:/# ls 
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@fdc372caedf7:/# cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.3 LTS"
```

-rm Automatically remove the container when its program exits.

```bash
docker container run --rm ubuntu:latest
```

--name string - Assign a name to  a docker container

```bash
docker container run --name my_webserver nginx
```

##### Exposing & Publishing  Container Ports

-d  run as a background process

```bash
[root@svr1 ~]# docker container run -d nginx
aac2d187d875c571ab7461614cad5b64a6453c6083fd4761c3e9e564e5517fc6
[root@svr1 ~]# curl http://172.25.10.50
curl: (7) Failed connect to 172.25.10.50:80; Connection refused
```
In here you can see, we can't access nginx welcome page even though nginx running.
The reason is, we didn't setup port mapping for nginx container. just only we did is expose port 3000.

--expose open up port 3000

```bash
[root@svr1 ~]# docker container run -d --expose 3000 nginx
1999e3a8a32c3f73e0a03663372141093ca096c712d1de76da609702474a242d
[root@svr1 ~]# docker container ls 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
1999e3a8a32c        nginx               "nginx -g 'daemon of…"   10 seconds ago      Up 9 seconds        80/tcp, 3000/tcp    nice_lewin
8317663311f0        nginx               "nginx -g 'daemon of…"   46 minutes ago      Up 46 minutes       80/tcp              hardcore_leakey
```

Now, I'm  going to  expose & map the port same time.


```bash
[root@svr1 ~]# docker container run -d --expose 3000 -p 8080:80 nginx
14a093de7fe4474bad8bdbf9a6480c4932b1e10aa8cded51528ee4d0a191e5c4
[root@svr1 ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                            NAMES
14a093de7fe4        nginx               "nginx -g 'daemon of…"   5 seconds ago       Up 4 seconds        3000/tcp, 0.0.0.0:8080->80/tcp   infallible_ellis
[root@svr1 ~]# curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

As same as above, Now I' forwarding nginx container default port 80 into host port 8018.

```bash
docker container run -d -p <HOST_PORT>:<CONTAINER_PORT> <IMAGE_ID>
```

```bash
root@svr1 ~]# docker container run -d -p 8081:80 nginx:latest 
df99bbd4a46acb5acb08a70f1102296c836f0db28c745f6ba23bc0b4b5a43d56
[root@svr1 ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
df99bbd4a46a        nginx:latest        "nginx -g 'daemon of…"   6 seconds ago       Up 5 seconds        0.0.0.0:8081->80/tcp   goofy_swanson
[root@svr1 ~]# curl localhost:8081
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

List All Port Mappings:

To see all the port mapping  for  a specific docker container.

```bash
[root@svr1 ~]# docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
df99bbd4a46a        nginx:latest        "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes        0.0.0.0:8081->80/tcp   goofy_swanson
[root@svr1 ~]# docker port df99bbd4a46a
80/tcp -> 0.0.0.0:8081
```

############# EXECUTE COMMANDS ON A CONTAINER ##################

There Three Ways...

1. Dockerfile

At the end of the Dockerfile for the nginx has mentioned "CMD"


2. During a Docker run


```bash
[root@svr1 ~]# docker container run -it nginx /bin/bash
root@509e7ea33288:/# pwd
/
root@509e7ea33288:/# ls /etc/nginx/conf.d/
default.conf
root@509e7ea33288:/# ls /etc/nginx/                    
conf.d/         koi-utf         mime.types      nginx.conf      uwsgi_params    
fastcgi_params  koi-win         modules/        scgi_params     win-utf         
root@509e7ea33288:/# ls /etc/nginx/
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf
```


3. Using the "exec" command







































