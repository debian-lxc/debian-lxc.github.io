---
published: true
layout: post
title: Creating Unprivileged Jessie Container
---

To use unprivileged jessie container on jessie host, you need to create special cgroup for the user, and change the container's init from systemd to sysvinit.

Notes on code snippets:
- Code started with ``host#`` are executed on the host as root. 
- Code started with ``host$`` are executed on the host as normal user (non-root). 
- Code started with ``c1#`` are executed on the container as root. 

## (Article in Progress)

## Prequisite
 
Latest stable version of lxc [installed](/installation.html)

## Allow normal user to use veth with bridge

Edit ``/etc/lxc/lxc-usernet``, add appropriate line for the user. By default lxc creates ``lxcbr0`` with NAT enabled, so you can just use that for the bridge.

```
# USERNAME TYPE BRIDGE COUNT
user    veth    lxcbr0  10
```

## Create Special cgroup for the user

Create a simple init script on ``/etc/init.d/user-cgroup``

```
#!/bin/bash
### BEGIN INIT INFO
# Provides:          user-cgroup
# Required-Start:    cgmanager
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: Create user-owned cgroups
### END INIT INFO

PATH="/sbin:/bin:/usr/sbin:/usr/bin"

. /lib/lsb/init-functions

# These variables can be defined on /etc/default/user-cgroup
# Top-level cgroup
PARENT="users"
# List of user names to create user-owned cgroup
# Each user will own /$PARENT/$USER cgroup
USERS="user"

if [ -f /etc/default/user-cgroup ]; then
        . /etc/default/user-cgroup
fi

case "$1" in
  start) ;;
  stop|restart|force-reload) exit 0 ;;
  *) echo "Usage: $0 {start|stop|restart|force-reload}" >&2; exit 1 ;;
esac

cgm movepidabs all / $$
cgm create all $PARENT
for USERNAME in $USERS;do
  cgm create all $PARENT/$USERNAME
  cgm chown all $PARENT/$USERNAME $(id -u $USERNAME) $(id -g $USERNAME)
done
```

Run it, check the result, and then set it to run automatically on boot. All user listed in ``USERS`` variable in the above script (or ``/etc/default/user-cgroup``) should have their own cgroup under ``users``.

```
host# invoke-rc.d user-cgroup start

host# cgm movepidabs all / $$

host# cgm listchildren cpu
users

host# cgm listchildren cpu users
user

host# update-rc.d user-cgroup defaults

host# ls /etc/rc*/*user-cgroup
/etc/rc2.d/S02user-cgroup  /etc/rc3.d/S02user-cgroup  /etc/rc4.d/S02user-cgroup  /etc/rc5.d/S02user-cgroup
```

## Modify user profile to use the cgroup on login

Edit user's ``$HOME/.profile``, add these lines at the end

```
# move to root-created user-owned cgroup
cgm movepidabs all /users/$USER $$
```

Logout, and then login. Check the result, you should be on ``users/$USER`` cgroup

```
host$ echo $USER
user
host$ cat /proc/self/cgroup
10:name=systemd:/users/user
9:perf_event:/users/user
8:net_prio:/users/user
7:net_cls:/users/user
6:freezer:/users/user
5:devices:/users/user
4:cpuset:/users/user
3:cpuacct:/users/user
2:cpu:/users/user
1:blkio:/users/user
```

## Create default container config for the user


At minimum, you need to copy entries from ``/etc/lxc/default.conf`` and add id mappings. Get correct id mappings from ``/etc/subuid`` and ``/etc/subgid``

```
host$ grep $USER /etc/subuid /etc/subgid
/etc/subuid:user:624288:65536
/etc/subgid:user:624288:65536
```

Create default container config

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

## Install LXC using Download Template
 
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

Don't start the container just yet. We need to set root password, replace systemd with sysvinit, and install sudo.

```
host$ lxc-usernsexec -- /usr/sbin/chroot ~/.local/share/lxc/c1/rootfs passwd

host$ lxc-usernsexec -- /usr/sbin/chroot ~/.local/share/lxc/c1/rootfs apt-get update

host$ lxc-usernsexec -- /usr/sbin/chroot ~/.local/share/lxc/c1/rootfs bash -c "PATH=/sbin:/usr/sbin:$PATH apt-get install sysvinit-core sudo"
```

## Start Container

```
host$ lxc-start -n c1
```

If you want to view the boot progress, you can immediately attach to ``tty0`` after starting the container. 

```
host$ lxc-start -n c1; lxc-console -n c1 -t 0
```

You can detach ``lxc-console`` using ``Ctrl-a q``. Check the container status from the host (using another session, or detach the ``lxc-console`` first)

```
host$ lxc-ls -f
NAME  STATE    IPV4       IPV6  GROUPS  AUTOSTART
-------------------------------------------------
c1    RUNNING  10.0.3.37  -     -       NO
```

## Accessing the Container

First way is with ``lxc-console``. You might need to press ``Enter`` to get login prompt to show.

```
host$ lxc-console -n c1

Connected to tty 1
Type <Ctrl+a q> to exit the console, <Ctrl+a Ctrl+a> to enter Ctrl+a itself

Debian GNU/Linux 8 c1 tty1

c1 login:
```

Second way is with ``lxc-attach``. Since this is unprivileged container, you also need to setup some environment variables (e.g. PATH, HOME). The easiest way is to combine ``lxc-attach`` with ``sudo -i``. You can ignore the warning about ``/dev/pts/0``.

```
host$ lxc-attach -n c1 -- sudo -i
mesg: /dev/pts/0: Operation not permitted
root@c1:~#
```



## Recommended: install ssh server and text editor

Run this command inside the container

```
c1# apt-get install openssh-server vim
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
