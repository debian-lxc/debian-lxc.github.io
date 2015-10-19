---
published: true
layout: post
title: Creating Privileged Jessie Container
---

The process is straightforward using the download template.

## Code convention

- Code started with ``host#`` are executed on the host as root. 
- Code started with ``host$`` are executed on the host as normal user (non-root). 
- Code started with ``c1#`` are executed on the container as root. 

## Prequisite

Latest stable version of lxc [installed](/installation.html)

## Create container using Download Template

In this example, the container name is ``c1``

```
host# lxc-create -n c1 -t download -- -d debian -r jessie -a amd64
Setting up the GPG keyring
Downloading the image index
Downloading the rootfs
Downloading the metadata
The image cache is now ready
Unpacking the rootfs

---
You just created a Debian container (release=jessie, arch=amd64, variant=default)

To enable sshd, run: apt-get install openssh-server

For security reason, container images ship without user accounts
and without a root password.

Use lxc-attach or chroot directly into the rootfs to set a root password
or create user accounts.
```

## Set root password using chroot

```
host# chroot /var/lib/lxc/c1/rootfs passwd
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

## Start Container

```
host# lxc-start -n c1
```

If you want to view the boot progress, you can immediately attach to ``tty0`` after starting the container. 

```
host# lxc-start -n c1; lxc-console -n c1 -t 0
```

Check the container status from the host (using another session, or detach the ``lxc-console`` first)

```
host# lxc-ls -f
NAME  STATE    IPV4        IPV6  GROUPS  AUTOSTART
--------------------------------------------------
c1    RUNNING  10.0.3.234  -     -       NO
```

## Accessing the Container

There are two ways to access the container that doesn't involve network:

- ``lxc-console``. You might need to press ``Enter`` to get login prompt to show. You can detach a running ``lxc-console`` using ``Ctrl-a q``. 

```
host$ lxc-console -n c1

Connected to tty 1
Type <Ctrl+a q> to exit the console, <Ctrl+a Ctrl+a> to enter Ctrl+a itself

Debian GNU/Linux 8 c1 tty1

c1 login:
```

- ``lxc-attach``. 

```
host$ lxc-attach -n c1
root@c1:~#
```

## Recommended: install ssh server and text editor

- Run this command inside the container

```
c1# apt-get update
c1# apt-get install openssh-server vim
```

- Optionally, allow root login with ssh using password. Edit ``/etc/ssh/sshd_config``, change

```
PermitRootLogin without-password
```

to

```
PermitRootLogin yes
```

- Restart ssh server

```
c1# invoke-rc.d ssh restart
```
