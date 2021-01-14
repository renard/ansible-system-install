<!--

---
lang: american
---
-->

# System install stage 2

This stage runs tasks on target system within the chroot environment.

## Description

For Debian / Ubuntu system tasks are:

* Disable `systemd-networkd` to use traditional `ifupdown` tools.
* Disable `systemd-resolved` to use traditional `resolvconf` tools.

See reasons in tasks files.
