---
published: true
layout: post
title: Update Systemd
---

Debian Jessie's version of systemd is not suitable to run unprivileged containers. To get the best experience, you need to replace systemd with Ubuntu's version.

Ubuntu's version of systemd has the following things:

- It contains a patch which automatically create a user-owned cgroup for every user on login on the host.
- It is recent-enough that is has fixes needed for some services (e.g. systemd-journald) to run correctly inside unprivileged container.

Use these steps to install Ubuntu Wily's version of systemd, ported to Jessie, along with its dependencies. Except for systemd, other packages were backported (e.g. apparmor) or copied (e.g. mount) from Stretch.

## Add Apt Source Entry

```
host# cat > /etc/apt/sources.list.d/debian-lxc-systemd.list << END
deb http://debian-lxc.github.io/packages/systemd jessie main
deb-src http://debian-lxc.github.io/packages/systemd jessie main
END
```

## Import public key

```
host# apt-key adv --recv-keys --keyserver keyserver.ubuntu.com C6E77093
```

## Check Available Packages

```
host# apt-get update
host# apt-cache policy systemd apparmor mount
```

## Install New Packages

```
host# apt-get install systemd-sysv libpam-systemd dbus
```

Reboot, login, then check the new systemd version. Each controller should have its own cgroup now.

```
host# systemd --version
systemd 225
+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ -LZ4 +SECCOMP +BLKID -ELFUTILS +KMOD -IDN

host# cat /proc/self/cgroup
8:devices:/user.slice/user-0.slice/session-9.scope
7:cpu,cpuacct:/user.slice/user-0.slice/session-9.scope
6:net_cls,net_prio:/user.slice/user-0.slice/session-9.scope
5:blkio:/user.slice/user-0.slice/session-9.scope
4:freezer:/user.slice/user-0.slice/session-9.scope
3:cpuset:/user.slice/user-0.slice/session-9.scope
2:perf_event:/user.slice/user-0.slice/session-9.scope
1:name=systemd:/user.slice/user-0.slice/session-9.scope
```