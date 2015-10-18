---
published: false
---

# Installation

## Install LXC

- Add apt source entry.
```bash
# cat > cat /etc/apt/sources.list.d/debian-lxc.github.io-lxc.list << END
deb http://debian-lxc.github.io/packages/lxc jessie main
deb-src http://debian-lxc.github.io/packages/lxc jessie main
END
```

- Import public key.
```bash
# apt-key adv --recv-keys --keyserver keyserver.ubuntu.com C6E77093
```

- Check available packages.
```bash
# apt-get update
# apt-cache policy cgmanager lxc lxcfs 
```

- Install LXC, forcing upgrade of related packages when necessary. 
```bash
# apt-get install cgmanager lxc lxc-templates lxcfs
```

## Enable Apparmor

- Edit existing kernel command line on ``/etc/default/grub`` to add apparmor-related entries. If you already have other entries present, add them at the end.
```
GRUB_CMDLINE_LINUX="apparmor=1 security=apparmor"
```

- Update grub menu
```bash
# update-grub
```

- Allow incomplete apparmor support. Edit ``/etc/lxc/default.conf``, add this line on top
```
lxc.aa_allow_incomplete = 1
```

- Reboot
- Verify apparmor status
```bash
# aa-status
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

# ls /sys/kernel/security/
apparmor
```