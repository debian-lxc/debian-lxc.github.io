---
published: true
layout: post
title: Installation
---


Use these steps to install lxc-1.1.4 on Debian Jessie

## Install LXC from This Repository

- Add apt source entry.

```
host# cat > /etc/apt/sources.list.d/debian-lxc.list << END
deb http://debian-lxc.github.io/packages/lxc jessie main
deb-src http://debian-lxc.github.io/packages/lxc jessie main
END
```

- Import public key.

```
host# apt-key adv --recv-keys --keyserver keyserver.ubuntu.com C6E77093
```

- Check available packages.

```
host# apt-get update
host# apt-cache policy cgmanager lxc lxcfs 
```

- Install LXC, forcing upgrade of related packages when necessary. 

```
host# apt-get install cgmanager lxc lxc-templates lxcfs
```

## Enable Apparmor

- Edit existing kernel command line on ``/etc/default/grub`` to add apparmor-related entries. If you already have other entries present, add them at the end.

```
GRUB_CMDLINE_LINUX="apparmor=1 security=apparmor"
```

- Update grub menu

```
host# update-grub
```

- Allow incomplete apparmor support. Edit ``/etc/lxc/default.conf``, add this line on top

```
lxc.aa_allow_incomplete = 1
```

- Reboot

- Verify apparmor status

```
host# aa-status
apparmor module is loaded.
4 profiles are loaded.
4 profiles are in enforce mode.
   /usr/bin/lxc-start
   lxc-container-default
   lxc-container-default-with-mounting
   lxc-container-default-with-nesting
0 profiles are in complain mode.
0 processes have profiles defined.
0 processes are in enforce mode.
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.

host# ls /sys/kernel/security/
apparmor
```

## Optional: Create init scripts for lxc-autostart and lxc-net

By default lxc package will install systemd services. If you replace ssytemd with sysvinit on the host, create these init scripts.

``/etc/init.d/lxc-net``

```
#!/bin/bash
### BEGIN INIT INFO
# Provides:          lxc-net
# Required-Start:    cgmanager
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: Auto create lxc bridge
### END INIT INFO

PATH="/sbin:/bin:/usr/sbin:/usr/bin"

. /lib/lsb/init-functions

case "$1" in
  start)
    /usr/lib/x86_64-linux-gnu/lxc/lxc-net start
    ;;
  stop)
    /usr/lib/x86_64-linux-gnu/lxc/lxc-net start
    ;;
  restart|force-reload) exit 0 ;;
  *) echo "Usage: $0 {start|stop|restart|force-reload}" >&2; exit 1 ;;
esac
```

``/etc/init.d/lxc-autostart``

```
#!/bin/bash
### BEGIN INIT INFO
# Provides:          lxc-autostart
# Required-Start:    lxc-net
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: Auto start user containers
### END INIT INFO

PATH="/sbin:/bin:/usr/sbin:/usr/bin"

. /lib/lsb/init-functions

case "$1" in
  start)
    /usr/lib/x86_64-linux-gnu/lxc/lxc-devsetup
    /usr/lib/x86_64-linux-gnu/lxc/lxc-apparmor-load
    /usr/lib/x86_64-linux-gnu/lxc/lxc-containers start
    ;;
  stop)
    /usr/lib/x86_64-linux-gnu/lxc/lxc-containers stop
    ;;
  restart|force-reload) exit 0 ;;
  *) echo "Usage: $0 {start|stop|restart|force-reload}" >&2; exit 1 ;;
esac
```

Also edit one line on ``/etc/init.d/lxcfs``

```
# Required-Start:       cgmanager
```

Activate and set them to automatically run on boot

```
host# invoke-rc.d lxcfs start

host# invoke-rc.d lxc-net start

host# invoke-rc.d lxc-autostart start

host# update-rc.d lxcfs remove

host# update-rc.d lxcfs defaults

host# update-rc.d lxc-net defaults

host# update-rc.d lxc-autostart defaults
```
