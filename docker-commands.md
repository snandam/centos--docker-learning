#### docker commands cheatsheet

##### Basic commands
```shell
# Check status of docker
service docker status
docker status

# Check version and info
docker version
docker info

# docker help
docker help
docker help inspect

# Search for an image in docker.io
docker search centos

# List number of images we have in local machine
docker images

# List containers
docker ps -a #Lists all containers
docker ps # Lists only running containers
```

##### Exercise 1 : Pull down an image and run some commands within a container

```shell
# Pull down an image from docker.io
docker pull ubuntu:latest

# Information on a container or image
docker inspect ubuntu

# To launch a container from an image and provide a command to run
# run options control the image’s runtime behavior in a container.
docker run -i -t ubuntu:latest /bin/bash
-i interactive bash prompt
-t current console (tty)

$ apt-get update
$ apt-cache search ruby
$ apt-get install ruby
# ruby --version

```
##### Notes :
- After you exit from the container and run the ubuntu:latest image whatever you had done to the container is not stored in ubuntu
- Changes has been stored to a different container

```shell

[root@localhost vagrant]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
centos              latest              ce20c473cd8a        4 days ago          172.3 MB
ubuntu              latest              a005e6b7dd01        5 days ago          188.3 MB

[root@localhost vagrant]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
c65a6df15314        ubuntu              "/bin/bash"         4 minutes ago       Exited (0) 4 seconds ago                          dreamy_wright
7bcd72c1b578        ubuntu              "/bin/bash"         9 minutes ago       Exited (127) 6 minutes ago                        angry_lumiere
7e5ab330f5a5        ubuntu              "/bin/bash"         10 minutes ago      Exited (1) 9 minutes ago                          elated_colden
afc808ec3e6a        centos:latest       "/bin/bash"         24 minutes ago      Exited (127) 24 minutes ago                       condescending_almeida
a4b15f84e20b        ubuntu              "/bin/bash"         34 minutes ago      Exited (1) 27 minutes ago
```

##### Exercise 2 : Save a container to an image with a proper name and tag and load the same
```shell
# Commit/create an image from container's changes
docker help commit
[root@localhost vagrant]# docker commit -m="install ruby to ubuntu base image" c65a6df15314 snandam/ubunturuby:v1.0
445f9a2ae01846bd7943b9e11e9b1b8f4b0adb16838e77ed1721a2b81d8ee9a4

[root@localhost vagrant]# docker images
REPOSITORY           TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
snandam/ubunturuby   v1.0                445f9a2ae018        About a minute ago   227 MB
centos               latest              ce20c473cd8a        4 days ago           172.3 MB
ubuntu               latest              a005e6b7dd01        5 days ago           188.3 MB

# docker run -t -i snandam/ubunturuby:v1.0 /bin/bash

```

##### Exercise 3 : Install puppet-lint gem to ubuntulibrarianpuppet:v1.0 and create a new image
```sh
# Launch a container
docker run -it snandam/ubuntulibrarianpuppet:v1.0 /bin/bash
# Make changes
root@3f22ebbb451a:/# gem install puppet-lint
# Commit to a new version
docker commit 3f22ebbb451a snandam/ubuntulibrarianpuppet:puppetlint
```

##### Exercise 4 : Automate and create a repeatable process to create a new image
###### Install librarian-puppet with ubuntu:latest as base image and save the new image with a different name

```shell
#Create a dockerfile
vi Dockerfile

# Contents
FROM ubuntu:latest
MAINTAINER gunmetalz
RUN apt-get update
RUN apt-get install ruby ruby-dev make git build-essential puppet -y
RUN gem install librarian-puppet

# Build a new image
docker build -t="snandam/ubuntulibrarianpuppet:v1.0" .
-t Repository name (and optionally a tag) for the image

# If it's successfully built you may see the images in the list

[root@localhost ubuntu_ruby]# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
snandam/ubuntulibrarianpuppet   v1.0                8e3efb31a5e8        16 seconds ago      429.1 MB
```

##### Exercise 5 : Run a command w/in a container
```shell
docker run "snandam/ubuntulibrarianpuppet:v1.0" /usr/bin/ruby "--version"
```

##### Exercise 6 : Run container as a daemon, view the output from container, stop, restart, pause and unpause
```shell
docker run -d "snandam/ubuntulibrarianpuppet:v1.0" /bin/bash -c "while true; do /usr/bin/ruby --version;sleep 1;done"
#-d Run container in background and print container ID
```


```
# List the running containers

[root@localhost ubuntu_ruby]# docker ps
CONTAINER ID        IMAGE                                COMMAND                CREATED             STATUS              PORTS               NAMES
cf0ae16d41d4        snandam/ubuntulibrarianpuppet:v1.0   "/bin/bash -c 'while   4 minutes ago       Up 4 minutes                            modest_hypatia

# View the stdout from the container
docker logs modest_hypatia

# Stop the container
docker stop modest_hypatia

# Restart a stopped container
docker restart modest_hypatia

# Kill the container
docker kill modest_hypatia
```
Notes:
- If the command fails or exits, the container won't be listed in the running containers.
- However the logs will be available for the command for the containerID

##### Exercise 7 : Create a container with name "test_container"
```sh
docker create -it --name="test_container" centos:nginx /bin/bash
docker start test_container
docker attach test_container

docker run -it --name="mycontainer" centos:nginx /bin/bash

```

##### Exercise 8 : Pull an nginx container and forward the host port to container port

```sh
#https://hub.docker.com/_/nginx/
docker run -d -p 3000:80 nginx:latest
#-p Publish a container's port(s) to the host


# Test out if that works
curl http://localhost:3000
```

##### Exercise 9 : Create a new image from centos:latest with nginx installed and accessible on port 1000 on local host
```sh
# Launch a new container
docker run -it centos:latest /bin/bash
# Install nginx
yum install epel-release -y
yum install nginx -y
/usr/sbin/nginx
curl http://localhost
# Commit to a new image
docker commit -m "Installing nginx and starting" 18618855156b centos:nginx

# Run the container as daemon
docker run -itd -p 1000:80 centos:nginx /bin/bash

# Attach to container
docker attach a2bf0ec5f19b

# Start nginx on the container and verify
[root@a2bf0ec5f19b /]# /usr/sbin/nginx
[root@a2bf0ec5f19b /]# curl http://localhost

# From another shell on host machine find out the ipaddress of the container, hit nginx
docker inspect a2bf0ec5f19b | grep IPAddress
# You can use --format or -f to query for specific output
docker inspect -f {{.NetworkSettings.IPAddress}} a2bf0ec5f19b
curl http://172.17.0.55
curl http://localhost:1000
```



##### Exercise 10 : Remove images from the local machine
```sh
# Remove image
docker rmi 38243d228ab1
# Remove container
docker rm b5fcfde033b1
# Remove all containers and restart docker
cd /var/lib/docker/containers
service docker restart

```
Notes:
- Containers would be using the base image that need to be deleted first
- Try not to orphan containers
- Containers are stacked up on each other when changes are made on top
- Loction of docker files - /var/lib/docker
- Tool to read json files - python -mjson.tool
- Where does docker images gets it's list from ?

```sh
[root@localhost docker]# cat /var/lib/docker/repositories-devicemapper | python -mjson.tool
```
- How do I find more information on each container? Look at the config file for each container
/var/lib/docker/containers


##### Exercise 11 : Create a new image from centos:nginx and make nginx start automatically when the container is launched
```sh
docker run -it centos:nginx /bin/bash
# Update .bashrc to run nginx or a different shell script to invoke /usr/sbin/nginx
vi /root/.bashrc
# Commit the container to new image
# Launch a container from the new image as daemon
docker run -itd -p 1000:80 centos:nginx_autostart /bin/bash
lynx http://localhost:1000

```
##### Exercise 12 : Push the centos:nginx_autostart box to docker hub after renaming the image
```sh
[root@localhost ~]# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
centos                          nginx_autostart     53748122ca15        37 minutes ago      316.6 MB
# Before pushing a repo format should be username/repo, so you can either tag an image with the new name or create a new image by running the image and committing it to new image

[root@localhost ~]# docker tag centos:nginx_autostart gunmetalz/centos:nginx_autostart
[root@localhost ~]# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
gunmetalz/centos                nginx_autostart     53748122ca15        41 minutes ago      316.6 MB
[root@localhost ~]# docker push gunmetalz/centos:nginx_autostart
```

##### Exercise 13 : Mount a volume or drive from local machine to docker container
```sh
docker run -i -t -v /opt/docker:/opt centos:nginx_autostart --name mycontainer /bin/bash
-v Bind mount a volume. sourcemachine:targetcontainer

docker inspect -f '{{ json .Mounts }}' mycontainer | jq


# /opt/docker is directory in local machine and /opt mount is created in the container
```

#### Exercise 14 : How do you find out what ip address range docker containers are assigned. Create a bridge adaptor for docker to use which is different than the default one

[root@localhost docker]# ifconfig
docker0   Link encap:Ethernet  HWaddr AE:A9:6B:F1:D4:DD
          inet addr:172.17.42.1  Bcast:0.0.0.0  Mask:255.255.0.0

#### Exercise 15 : Launch a container of centos:nginx_autostart as daemon. Make sure nginx is running without attaching to the container. Execute a command on the container
```sh
docker run -itd --name="mycontainer" centos:nginx_autostart /bin/bash
docker exec -i -t mycontainer /bin/bash -c "top"
# To run the command and exits
docker top mycontainer

```

#### Exercise 16 : Remove containers that are older than certain days or remove all of them
```sh
docker ps -a | grep '7 days ago' | awk '{print $1}' | xargs docker rm
docker ps -a -q | xargs docker rm
```
#### Exercise 17 : Expose container outside the vagrant machine to local system
```
- To set a static route on the system I want to access
- Find out the ip range docker containers are running and the ip address of the virtual machine
[root@localhost docker]# ifconfig
docker0   Link encap:Ethernet  HWaddr 0E:B2:33:AA:E4:64
          inet addr:172.17.42.1  Bcast:0.0.0.0  Mask:255.255.0.0

eth1      Link encap:Ethernet  HWaddr 08:00:27:8D:A0:52
          inet addr:192.168.50.4  Bcast:192.168.50.255  Mask:255.255.255.0


- Add route on the machine you want the container access
route add -net 172.17.0.0 netmask 255.255.255.0 gw 192.168.50.4
- gw gateway which at this point is the ipaddress of the host the container is running on


# Add and remove static route in mac
sudo route add -net 172.17.0.0/23 192.168.50.4
 netstat -rn

 Internet:
 Destination        Gateway            Flags        Refs      Use   Netif Expire
 172.17/23          192.168.50.4       UGSc            1        0 vboxnet

sudo route delete -net 172.17.0.0/23 192.168.50.4

```
#### Exercise 18 : Spin up two containers and share volumes between two of them. Instead of sharing a local drive to two containers to avoid traffic to come to local machine

```sh

docker run -itd  -v --name="NGINX1" /mymount centos:nginx /bin/bash

# Inherit the mount from the previous container

docker run -itd  --volumes-from NGINX1 --name="NGINX2" centos:nginx /bin/bash
# /mymount will be shared between the two containers

```

#### Exercise 19 : link a previous container to a new container

```sh
docker run -it --name "STATIC" --link NGINX2:localweb centos:nginx /bin/bash
```

#### Exercise 20 : If multiple containers are running on the same port, how to handle redirects
```sh
[root@localhost docker]# docker run -itd -P --name="NGINX1" centos:nginx_autostart /bin/bash
73db572095b867fc2f9bcf963a57df0d16ba4adda1faffb076ce7d58f7eb3a50
[root@localhost docker]# docker ps
CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                   NAMES
73db572095b8        centos:nginx_autostart   "/bin/bash"         2 seconds ago       Up 2 seconds        0.0.0.0:32768->80/tcp   NGINX1   
- P : capital P is to map any exposed post inside the container to a random port in the local system. Between 49000 and 49900

# Map to any port for 127.0.0.1 with the container port 80

docker run -itd -p 127.0.0.1::80 --name="NGINX10" centos:nginx_autostart /bin/bash
```

#### Exercise 21 : Copy a file from the container to /tmp

```sh
docker cp containerID:/etc/yum.repos.d /tmp

```

#### Exercise 22 : What was changed to the container after it was launched
```sh
docker diff NGINX1
```

#### Exercise 23 : How to find all the docker events that happened in the system

```sh
docker events
docker events --since '2015-09-05'
# On an image
docker history centos:nginx_autostart
```

#### Exercise 24 : Share/Export a container

```sh
docker export NGINX1 > nginx.tar

```
- Inorder for this to work an image need to have the port exposed when building the image



#### Exercise 25 : Create a bridge network and add container to that network

```sh
docker network create -d bridge web-bridge
docker network ls
docker run -d --network=web-bridge -P --name web training/webapp python app.py
docker network inspect web-bridge
docker inspect -f {{.HostConfig.NetworkMode}} web
docker inspect --format='{{json .NetworkSettings.Networks}}' web | jq
```

#### Exercise 26 : Run a container for db and join the container to the same network

```sh
docker run -d --name db training/postgres
# By default all the containers connects to the default bridge network. These two containers can't talk to
# each other unless they are in the same network
docker network connect web-bridge db
```

#### Exercise 27: Create a docker volume of 25 MB and mount the volume to a container


```sh
docker volume create --opt o=size=20MB --name my-named-volume


```
##### Notes :
- Q: Why use -i and -d in the docker run command. If the docker is running a daemon and i is to run interactive why give at the same time?
- A: By running with -i and /bin/bash you can attach to the container to it's shell prompt.



###### Docker file commands

```vi
FROM centos:latest
MAINTAINER gunmetalz

EXPOSE 22 80
RUN yum install epel-release -y
ADD fileincurrentdir.html /var/nginx/www/html



# TO run one command instead of running one at a time. Takes less time to build

RUN yum install nginx -y && yum install wget -y
```
