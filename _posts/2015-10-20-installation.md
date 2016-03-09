---
published: true
layout: post
title: Installation
---

Use these steps to install lxc-1.1.5 on Debian Jessie

## Code convention

- Code started with ``host#`` are executed on the host as root. 

## Install LXC

- Add apt source entry.

```
host# cat > /etc/apt/sources.list.d/debian-lxc-lxc.list << END
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

## Enable Apparmor and Memory Cgroup Support

- Edit existing kernel command line on ``/etc/default/grub`` to add these entries. If you already have other entries present, add them at the end.

```
GRUB_CMDLINE_LINUX="apparmor=1 security=apparmor cgroup_enable=memory swapaccount=1"
```

- Update grub menu

```
host# update-grub
```

- Allow incomplete apparmor support. Edit ``/etc/lxc/default.conf``, add this line on top

```
lxc.aa_allow_incomplete = 1
```

- Enable ``unprivileged_userns_clone``. Needed for unprivileged containers.

```
host# cat > /etc/sysctl.d/50-lxc-unprivileged.conf << END
kernel.unprivileged_userns_clone = 1
END

host# sysctl --system -p
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

## Optional: sysvinit or systemd

- If you replace sytemd with sysvinit on the host, you need to [create init scripts manually](/Create-Init-Scripts.html)

- If you keep systemd on the host, and want to easily [use unprivileged containers](/Create-Unprivileged-Jessie-Container.html), you might want to [use Ubuntu-patched systemd](/Update-Systemd.html). Alternatively, you can also use unprivileged containers with Debian's version of systemd if you [create user-owned cgroup](/Create-User-Owned-Cgroup.html) manually.

