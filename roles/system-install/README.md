<!--

---
lang: american
---
-->

# System install

This Ansible role installs a Linux system on a remote host. It can
install [Debian](https://debian.org), [Ubuntu](https://ubuntu.com/),
[CentOS](https://www.centos.org/), [Fedora](https://getfedora.org/),
[OpenSUSE](https://www.opensuse.org/),
[Alpine](https://alpinelinux.org/), and
[Archlinux](https://www.archlinux.org/) distributions.

## Prerequisite

You will need to boot your server using a live CD or use a PXE boot
that let your hard drive unused. This role has been designed to
against run against a recent Debian (Buster) system (understand a
machine booted with a Debian live CD). It may also work from older
Debian live CD or with Ubuntu live CD.

You can find live images on following sites:

- https://www.debian.org/CD/live/
- http://www.ubuntu.com/download/desktop

With Debian live images *ssh* is not installed by default. You have to log
in and run following commands:

```shell
sudo -s
apt-get update
apt-get install openssh-server
/etc/init.d/ssh restart
```

If you are using VirtualBox you can run a script like:

```shell
VBoxManage controlvm TARGET_VM keyboardputstring \
	   "sudo bash -c 'apt-get update; apt-get install -y openssh-server; /etc/init.d/ssh restart;ip a l'"
VBoxManage controlvm TARGET_VM keyboardputscancode 1c 9c
```


If you are using Private Server hosting you should reboot the server in
rescue mode. This might also work with some Virtual Private Server
provider. Please check your provider support.


## Role Description

The install process runs in several parts:

- Remove all partitions and create a new partition scheme (see
  `system_install_partitions` variable).
- Install deployment tools (such as *debootstrap*, *yum*, *apk*...)
  depending on the target system.
- Bootstrap the system and install additional packages (see
  `system_install_additional_packages`).
- Fix *ssh* configuration to allow **root** login (`PermitRootLogin` is set
  to `yes`).
- Update *grub* configuration to be more verbose and install it on the hard
  drive.


### Security warning

Setting up a root password and allowing *ssh* access using this method is
not secure. Please consider using ssh keys and do not forget to secure the
system after the reboot from hard drive.

## Configuration

See specific documentation in defaults files in [defaults/main]().

## Extra

If you want to create an image of you newly installed system for
future usage, you can run `/usr/local/bin/generate-base-image` as
root. The archive is not minimalistic but is a good start.


## Tested versions


| Version                | Status | EOL                            | Restrictions      |
|------------------------|--------|--------------------------------|-------------------|
| Debian 8 (jessie)      | OK     | 2020-06                        | [^py2] [^xfs]     |
| Debian 9 (stretch)     | OK     | 2022-06                        | [^xfs]            |
| Debian 10 (buster)     | OK     | 2024                           |                   |
| Debian 11 (bullseye)   | OK     | TBA                            |                   |
| Debian 12 (bookworm)   | N/A    | TBA                            | Not yet available |
|                        |        |                                |                   |
| Ubuntu 14.04 (trusty)  | OK     | LTS: 2019-04-30 / EMS: 2022-04 | [^py2] [^xfs]     |
| Ubuntu 14.10 (utopic)  |        | 2015-07-23                     | Do not use        |
| Ubuntu 15.04 (vivid)   |        | 2016-02-04                     | Do not use        |
| Ubuntu 15.10 (wily)    |        | 2016-07-28                     | Do not use        |
| Ubuntu 16.04 (xenial)  | OK     | LTS: 2021-04 / EMS: 2024-04    | [^py2] [^xfs]     |
| Ubuntu 16.10 (yakkety) |        | 2017-07-20                     | Do not use        |
| Ubuntu 17.04 (zesty)   |        | 2018-01-13                     | Do not use        |
| Ubuntu 17.10 (artful)  |        | 2018-07-19                     | Do not use        |
| Ubuntu 18.04 (bionic)  | OK     | LTS: 2023-04 / EMS: 2028-04    | [^xfs]            |
| Ubuntu 18.10 (cosmic)  |        | 2018-10-18                     | Do not use        |
| Ubuntu 19.04 (disco)   |        | 2020-01-23                     | Do not use        |
| Ubuntu 19.10 (eoan)    |        | 2020-07-17                     | Do not use        |
| Ubuntu 20.04 (focal)   | OK     | LTS: 2025-04 / EMS: 2030-04    |                   |
| Ubuntu 20.10 (groovy)  |        | 2021-07                        |                   |
|                        |        |                                |                   |
| CentOS 7               | OK     | 2024-06-30                     |                   |
| CentOS 8               | OK     | 2029-05-31                     |                   |
|                        |        |                                |                   |
| Fedora 31              | OK     | N/A                            |                   |
| Fedora 32              | OK     | N/A                            |                   |
| Fedora 33              | N/A    | N/A                            |                   |
|                        |        |                                |                   |
| OpenSUSE 15.1          | OK     | 2020-11-22                     |                   |
| OpenSUSE 15.2          | OK     | 2021-12-31                     |                   |
|                        |        |                                |                   |
| Alpine 3.9             | OK     | 2020-11-01                     |                   |
| Alpine 3.10            | OK     | 2021-05-01                     |                   |
| Alpine 3.11            | OK     | 2021-11-01                     |                   |
| Alpine 3.12            | OK     | 2022-05-01                     |                   |
|                        |        |                                |                   |
| Archlinux 2020.07.01   | OK     | N/A                            |                   |
| Archlinux 2020.08.01   | OK     | N/A                            |                   |
| Archlinux 2020.09.01   | OK     | N/A                            |                   |


[^py2]: See [vars/main.yml]() for python support in chroot.
[^xfs]: See [vars/main.yml]() for xfs options support.


## Copyright

Author: Sébastien Gross

License: WTFPL, grab your copy here: http://www.wtfpl.net
