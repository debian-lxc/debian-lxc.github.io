# Debian LXC

A series of HOWTOs on how to use latest stable version of [Linux Containers](https://linuxcontainers.org) on Debian, including unprivileged containers.

[Web Page](http://debian-lxc.github.io) created using [Lanyon](https://github.com/poole/lanyon)

## Contents

- [Usage](#usage)


## Usage

- Add apt source entry
```bash
# cat > cat /etc/apt/sources.list.d/debian-lxc.github.io-lxc.list << END
deb http://debian-lxc.github.io/packages/lxc jessie main
deb-src http://debian-lxc.github.io/packages/lxc jessie main
END
```

- Import public key
```bash
# apt-key adv --recv-keys --keyserver keyserver.ubuntu.com C6E77093
```

- Check available packages
```bash
# apt-get update
# apt-cache policy cgmanager lxc lxcfs 
```

- Install LXC
```bash
# apt-get install cgmanager lxc lxc-templates lxcfs
```
