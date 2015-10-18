---
published: true
layout: post
title: Creating Unprivileged Jessie Container
---


To create unprivileged container, you need to create special cgroup for the user, and change the container's init from systemd to sysvinit

## (Article in Progress)

## Prequisite
 
Latest stable version of lxc [installed](/installation.html)
 
## Create Special cgroup for the user

todo

## Install LXC using Download Template
 
Commands with ``host$`` are executed on the host as normal user (non-root). In this example, the container name is ``c1``, and the user name is ``user``
 
```
host$ lxc-create -n c1 -t download -- -d debian -r jessie -a amd64
```

## Post-Create Container Configuration using chroot and lxc-usernsexec

todo

## Start Container

todo

## Recommended: install ssh server and text editor

todo
