<!--
---
lang: american
---
-->

# System install stages

Setup is run in 3 stages:

* Install system (bootstrap or a tarball image).
* Customize the system (you can where you can add your own tasks).
* Cleanup the install and unmount all filesystems.

You need to reboot the remote server yourself.

## Stage 1: system bootstraping

This stage starts to create a filesystem for the future OS
installation using FAI's
[setup-storage](https://wiki.fai-project.org/index.php/Setup-storage). All
data will be removed. It tries to unmount existing partitions and free
hard drives, however if some resources such as RAID devices are still
in use this stage will fail.

If any RAID device is in use, you can do:

```
awk '{if($3=="active"){print "/dev/"$1}}' /proc/mdstat \
  | xargs --no-run-if-empty mdadm --stop
```

If any LVM device is in use, you can do:

```
vgchange -an
pvremove /dev/md0 --force --force
```

After filesystem creation the target OS in installed using different
tools depending on the OS to install (`debootstrap` for Debian based,
`yum` for CentOS or Fedora, `apk` for Alpine, `tar` for Archlinux...)
or using a pre-generated image.

Next some fix are made to ensure DNS resolution and Grub is installed
into all device marked as bootable before fixing a few things
depending on distribution (such as interface configuration files or
network activation or SELinux deactivation). If SELinux has to be
activated, please do it in stage 2 or use an other role for that.

Enventually a *ssh* server is started within the chroot for stage 2
tasks.

## Stage 2: Actions within the target chroot

Stage 2 allows to use Ansible tasks and roles directly to the chroot.
Since Ansible (at least version 2.9) cannot pipeline connections (such
as using chroot on a remote host) it uses a ssh server from the chroot
itself.

By default only a few things are done, mainly fixing *systemd* usage.

## Stage 3: Cleanup

Stage 3 purpose is to cleanup the installation process. It removes
root ssh access (configurable, see the defaults), stops the ssh server
within the chroot and unmount all partitions.

The system is ready to be rebooted to the new installed OS.
