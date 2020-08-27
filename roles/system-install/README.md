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

## Copyright

Author: Sébastien Gross

License: WTFPL, grab your copy here: http://www.wtfpl.net
