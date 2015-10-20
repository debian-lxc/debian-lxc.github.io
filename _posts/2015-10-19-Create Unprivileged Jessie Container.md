---
published: true
layout: post
title: Create Unprivileged Jessie Container
---

Using unprivileged jessie container on jessie host is possible, but requires some additional setup compared to privileged containers. 

In summary, you need to:

- have uid & gid mappings assigned to the user. Check ``/etc/subuid`` and ``/etc/subgid``
- allow the user to use veth with bridge
- create special cgroup for the user
- change the container's init from systemd to sysvinit.

## Code convention

- Code started with ``host#`` are executed on the host as root. 
- Code started with ``host$`` are executed on the host as normal user (non-root). 
- Code started with ``c1#`` are executed on the container as root. 

## Prequisite
 
- Latest stable version of lxc [installed](/installation.html)
- User-owned cgroup. This is created and assigned using either:
  - [init script and user profile](/Create User-Owned Cgroup.html), or
  - [pam_systemd](/Update Systemd.html)

## Allow normal user to use veth with bridge

Edit ``/etc/lxc/lxc-usernet``, add appropriate line for the user. By default lxc creates ``lxcbr0`` with NAT enabled, so you can just use that for the bridge.

```
# USERNAME TYPE BRIDGE COUNT
user    veth    lxcbr0  10
```

## Create default container config for the user


At minimum, you need to copy entries from ``/etc/lxc/default.conf`` and add id mappings. 

- Get correct id mappings from ``/etc/subuid`` and ``/etc/subgid``

```
host$ grep $USER /etc/subuid /etc/subgid
/etc/subuid:user:624288:65536
/etc/subgid:user:624288:65536
```

- Create default container config. Make sure the numbers on ``lxc.id_map`` matches what you see on ``/etc/subuid`` and ``/etc/subgid``

```
host$ mkdir -p ~/.config/lxc

host$ cat > ~/.config/lxc/default.conf << END
lxc.id_map = u 0 624288 65536
lxc.id_map = g 0 624288 65536
END

host$ cat /etc/lxc/default.conf >> ~/.config/lxc/default.conf

host$ cat ~/.config/lxc/default.conf
lxc.id_map = u 0 624288 65536
lxc.id_map = g 0 624288 65536
lxc.aa_allow_incomplete = 1
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:xx:xx:xx
```

## Create container using Download Template
 
In this example, the container name is ``c1``.
 
```
host$ lxc-create -n c1 -t download -- -d debian -r jessie -a amd64
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

## Post-Create Container Configuration using chroot and lxc-usernsexec

Don't start the container just yet. We need to set root password, replace systemd with sysvinit, install sudo, and fix pam settings bug.

```
host$ lxc-usernsexec -- /usr/sbin/chroot ~/.local/share/lxc/c1/rootfs passwd

host$ lxc-usernsexec -- /usr/sbin/chroot ~/.local/share/lxc/c1/rootfs apt-get update

host$ lxc-usernsexec -- /usr/sbin/chroot ~/.local/share/lxc/c1/rootfs bash -c "PATH=/sbin:/usr/sbin:$PATH apt-get install sysvinit-core sudo"

host$ lxc-usernsexec -- /usr/sbin/chroot ~/.local/share/lxc/c1/rootfs sed -i "s/required.*pam_loginuid.so/optional pam_loginuid.so/g" /etc/pam.d/login
```

## Start Container

```
host$ lxc-start -n c1
```

If you want to view the boot progress, you can immediately attach to ``tty0`` after starting the container. 

```
host$ lxc-start -n c1; lxc-console -n c1 -t 0
```

Check the container status from the host (using another session, or detach the ``lxc-console`` first)

```
host$ lxc-ls -f
NAME  STATE    IPV4       IPV6  GROUPS  AUTOSTART
-------------------------------------------------
c1    RUNNING  10.0.3.37  -     -       NO
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

- ``lxc-attach``. Since this is unprivileged container, you also need to setup some environment variables (e.g. PATH, HOME). The easiest way is to combine ``lxc-attach`` with ``sudo -i``. You can ignore the warning about ``/dev/pts/0``.

```
host$ lxc-attach -n c1 -- sudo -i
mesg: /dev/pts/0: Operation not permitted
root@c1:~#
```

## Recommended: install ssh server and text editor

- Run this command inside the container

```
c1# apt-get install openssh-server vim
c1# sed -i "s/required.*pam_loginuid.so/optional pam_loginuid.so/g" /etc/pam.d/sshd
```

Optionally, allow root login with ssh using password. Edit ``/etc/ssh/sshd_config``, change

```
PermitRootLogin without-password
```

to

```
PermitRootLogin yes
```

and then restart ssh server

```
c1# invoke-rc.d ssh restart
```
