---
published: true
layout: post
title: Creating Unprivileged Jessie Container
---

To create unprivileged container, you need to create special cgroup for the user, and change the container's init from systemd to sysvinit.

Code started with ``host#`` are executed on the host as normal user (non-root). 
Code started with ``host$`` are executed on the host as normal user (non-root). 
Code started with ``c1#`` are executed on the container as root. 

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

## Install LXC using Download Template
 
In this example, the container name is ``c1``.
 
```
host$ lxc-create -n c1 -t download -- -d debian -r jessie -a amd64
```

## Post-Create Container Configuration using chroot and lxc-usernsexec

todo

## Start Container

todo

## Recommended: install ssh server and text editor

todo
