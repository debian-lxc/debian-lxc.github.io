---
published: true
layout: post
title: Create User-Owned Cgroup
---

This is only needed if you don't use [Ubuntu-patched systemd on the host](/Update%20Systemd.html).

## Create Init Script 

- Create ``/etc/init.d/user-cgroup``

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

- Make it executable
 
```
host# chmod 755 /etc/init.d/user-cgroup
```

## Activate

- Update systemd to recognize the new init script (needed if you use systemd as init on the host, which is the default)

```
host# systemctl daemon-reload
```

- Run the service, check the result, and then set it to run automatically on boot. All user listed in ``USERS`` variable in the above script (or ``/etc/default/user-cgroup``) should have their own cgroup under ``users``.

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

## Modify User Profile to Use the Cgroup on Login

- Login as non-root user, edit ``$HOME/.profile``. Add these lines at the end

```
# move to root-created user-owned cgroup
cgm movepidabs all /users/$USER $$
```

- Logout, and then login again as non-root user. Check the result, you should be on ``users/$USER`` cgroup

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
