<!--

---
lang: american
---
-->

# System install

This Ansible role installs a Debian / Ubuntu on a remote host.

## Prerequisite

You will need to boot your server using a live CD or use a PXE boot that let
your hard drive unused.

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

- Install *debootstrap* and required tools.
- Remove all partitions and create a new partition scheme (see
  `system_install_partitions` variable).
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

See specific documentation in `defaults/main.yml`.

## Extra

If you want to create an image of you newly installed system for future
usage, you can run the following command as root:


```
tar --anchored --preserve-permissions --numeric-owner \
  --xattrs --xattrs-include '*' --selinux --acls --one-file-system \
  --exclude './**/lost+found/*' \
  --exclude './**/dead.letter' \
  --exclude './**/.bash_history' \
  --exclude './**/.ansible' \
  --exclude './proc/*' \
  --exclude './dev/*' \
  --exclude './sys/*' \
  --exclude './run/*' \
  --exclude './tmp/*' \
  --exclude './tmp/.*' \
  --exclude './var/tmp/*' \
  --exclude './var/log/**/*' \
  --exclude './var/log/*.gz' \
  --exclude './var/log/*.1' \
  --exclude './var/log/*log' \
  --exclude './var/log/*tmp' \
  --exclude './var/log/*.err' \
  --exclude './var/cache/**/*' \
  --exclude './var/spool/**/*' \
  --exclude './var/backups/*' \
  -C / \
  -czvf /tmp/`lsb_release -si | tr '[A-Z]' '[a-z]'`-`lsb_release -sc`-`dpkg --print-architecture`.tgz .
```

The archive is not minimalistic but is a good start. If you want to create
more optimized images for PXE usage have a look at the `ramdisk` role.


## Tested versions


| Version                | Status | EOL        | Restrictions      |
|------------------------|--------|------------|-------------------|
| Debian 8 (jessie)      | OK     | 2020-06    | (1) (2)           |
| Debian 9 (stretch)     | OK     | 2022-06    | (2)               |
| Debian 10 (buster)     | OK     | 2024       |                   |
| Debian 11 (bullseye)   | OK     | TBA        |                   |
| Debian 12 (bookworm)   | N/A    | TBA        | Not yet available |
|------------------------|--------|------------|-------------------|
| Ubuntu 16.04 (xenial)  | OK     | 2021-04    | (1) (2)           |
| Ubuntu 16.10 (yakkety) |        | 2017-07-20 |                   |
| Ubuntu 17.04 (zesty)   |        | 2018-01-13 |                   |
| Ubuntu 17.10 (artful)  |        | 2018-07-19 |                   |
| Ubuntu 18.04 (bionic)  | OK     | 2023-04    | (2)               |
| Ubuntu 18.10 (cosmic)  |        | 2018-10-18 |                   |
| Ubuntu 19.04 (disco)   |        | 2020-01-23 |                   |
| Ubuntu 19.10 (eoan)    |        | 2020-07-17 |                   |
| Ubuntu 20.04 (focal)   | OK     | 2025-04    |                   |
| Ubuntu 20.10 (groovy)  |        | 2021-07    |                   |
|------------------------|--------|------------|-------------------|
| CentOS 7               | OK     | 2024-06-30 |                   |
| CentOS 8               | OK     | 2029-05-31 |                   |
|------------------------|--------|------------|-------------------|
| Fedora 31              | OK     | N/A        |                   |
| Fedora 32              | OK     | N/A        |                   |
| Fedora 33              | N/A    | N/A        |                   |
|------------------------|--------|------------|-------------------|
| OpenSUSE 15.1          | OK     | 2020-11-22 |                   |
| OpenSUSE 15.2          | OK     | 2021-12-31 |                   |
|------------------------|--------|------------|-------------------|
| Alpine 3.9             | OK     | 2020-11-01 |                   |
| Alpine 3.10            | OK     | 2021-05-01 |                   |
| Alpine 3.11            | OK     | 2021-11-01 |                   |
| Alpine 3.12            | OK     | 2022-05-01 |                   |



(1) Python2 in chroot. Use `-e chroot_python_interpreter='/usr/bin/python'` option.

(2) XFS is not compatible between kernel 3.16 and 4.9
mkfs.xfs version 3.2.1
```
versionnum [0xb4a4+0x8a] = V4,NLINK,ALIGN,DIRV2,LOGV2,EXTFLG,MOREBITS,ATTR2,LAZYSBCOUNT,PROJID32BIT
meta-data=/dev/vdb               isize=256    agcount=4, agsize=6554 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=26214, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=853, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

mkfs.xfs version 4.20.0
```
xfs_db /dev/vdb -c version
versionnum [0xb4a5+0x18a] = V5,NLINK,DIRV2,ALIGN,LOGV2,EXTFLG,MOREBITS,ATTR2,LAZYSBCOUNT,PROJID32BIT,CRC,FTYPE,FINOBT,SPARSE_INODES

meta-data=/dev/vdb               isize=512    agcount=4, agsize=6554 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=0
data     =                       bsize=4096   blocks=26214, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=855, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```


## Copyright

Author: Sébastien Gross

License: WTFPL, grab your copy here: http://www.wtfpl.net
