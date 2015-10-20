---
published: true
layout: post
title: Create Init Scripts

Lxc package from this repository will install systemd service units. If you replace sytemd with sysvinit on the host, You need to create init scripts manually.

- Create ``/etc/init.d/lxc-net``

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

- Create ``/etc/init.d/lxc-autostart``

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

- Edit one line on ``/etc/init.d/lxcfs``

```
# Required-Start:       cgmanager
```

- Activate and set them to automatically run on boot

```
host# chmod 755 /etc/init.d/lxc-net /etc/init.d/lxc-autostart

host# invoke-rc.d lxcfs start

host# invoke-rc.d lxc-net start

host# invoke-rc.d lxc-autostart start

host# update-rc.d lxcfs remove

host# update-rc.d lxcfs defaults

host# update-rc.d lxc-net defaults

host# update-rc.d lxc-autostart defaults
```
