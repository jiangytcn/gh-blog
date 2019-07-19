---
layout: post
category : Docker
tagline: ""
tags : [Docker, Sysadmin]
---

This introduction will outline specifically  how to setup a private docker registry and how building a debian based docker image for later use.

## Overview
The following commands run on redhat 6.5

### Set up docker Registry
------------------------------------

Requirements
------------------
The Registry is compatible with Docker engine version 1.6.0 or higher.

TL;DR
--------
    # Start your registry
    docker run -d -p 5000:5000 registry:2
    after the command over, you can see a running container
    
![runing just as mine](http://7fvkn1.com1.z0.glb.clouddn.com/$722FED96F98230C5.jpg)

It works.

### Build Debian Base Image(Jessie)
-------------------------------------------------

####First install debootstrap

> sudo yum install debootstrap

####Then prepare the root directory
> mkdir -p /vdisks/chroot/jessie

####Finally build a Jessie system
> sudo debootstrap jessie /vdisks/chroot/jessie http://ftp.us.debian.org/debian
####Waiting for the result
####...
####At this point you have a small environment which you can chroot into and modify it the way you like
![build image](http://7fvkn1.com1.z0.glb.clouddn.com/$3A6B5AB4669806E9.jpg)

#### Import Image

    [yitao@yitao-dev ~]$ cd /vdisks/chroot/
    [yitao@yitao-dev chroot]$ sudo tar -C jessie/ -c . | sudo docker import - debian-jessie
    79911aebfadd3f5d4fb7a3b539ee658361909bea1e64d25385b5bbb97e4a1d0c

	Local images
    [yitao@yitao-dev ~]$ sudo docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
    debian-jessie       latest              79911aebfadd        About a minute ago   274.8 MB
    registry            2                   b4ad0b763f11        2 weeks ago          548.6 MB

### Upload Customize Image to Private Registry
-----------------------------------------------------------------
       
    # Tag the image so that it points to your registry
    sudo docker tag debian-jessie localhost:5000/debian:jessie
    
    # Push it
    [yitao@yitao-dev chroot]$ sudo docker push localhost:5000/debian:jessie
    The push refers to a repository [localhost:5000/debian] (len: 1)
    79911aebfadd: Image already exists 
    Digest: sha256:a7769244195e4230119e8753acd015f3e2804321eb21fcadb200910cb9d93394
    [yitao@yitao-dev chroot]$     
    
    # Pull it back 
    [yitao@yitao-dev ~]$ sudo docker pull localhost:5000/debian:jessie
    jessie: Pulling from localhost:5000/debian
    79911aebfadd: Already exists 
    Digest: sha256:a7769244195e4230119e8753acd015f3e2804321eb21fcadb200910cb9d93394
    Status: Image is up to date for localhost:5000/debian:jessie

    #start a new image
    [yitao@yitao-dev ~]$ sudo docker run --rm --name=jessie localhost:5000/debian:jessie cat /etc/issue.net
	Debian GNU/Linux 8

Thank you.