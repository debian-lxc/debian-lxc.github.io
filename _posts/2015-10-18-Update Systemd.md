By default, Debian Jessie's version of systemd is not suitable to run unprivileged containers. To get the best experience, you need to replace systemd with Ubuntu's version.

Ubuntu's version of systemd has the following things:

- It contains a patch which automatically create a user-owned cgroup for every user on login on the host.
- It is recent-enough that is has fixes needed for some services (e.g. systemd-journald) to run correctly inside unprivileged container.

Use these steps to install Ubuntu Wily's version of systemd, ported to Jessie, along with its dependencies. Except for systemd, other packages were backported (e.g. apparmor) or copied (e.g. mount) from Stretch.

## Add Apt Source Entry.

```
host# cat > /etc/apt/sources.list.d/debian-lxc-systemd.list << END
deb http://debian-lxc.github.io/packages/systemd jessie main
deb-src http://debian-lxc.github.io/packages/systemd jessie main
END
```

## Check Available Packages.

```
host# apt-get update
host# apt-cache policy systemd apparmor mount
```

## Install New Packages

```
host# apt-get install systemd-sysv systemd
```

Reboot, then check the new systemd version

```
host# systemd --version
systemd 225
+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ -LZ4 +SECCOMP +BLKID -ELFUTILS +KMOD -IDN
```