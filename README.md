# qemu-patched
[qemu](https://www.archlinux.org/packages/extra/x86_64/qemu/) from Arch Linux, PKGBUILD patched for anti-vm detection.
Repository forked from [nbaertsch/qemu-git-patched-pkgbuild](https://github.com/nbaertsch/qemu-git-patched-pkgbuild).

This along with the kernel patch and common-sense changes to libvirt xml clears all pafish checks.
Incorporates [dynamic hypervisor flag hiding by PiMaker](https://gist.github.com/PiMaker/70d01cc27792418e8e14e9b2b442129c) in CPUID for extra performance.

# Install
1. Clone
1. Change string values in ``qemu_hide_hv_qmp_disable.patch`` to be unique
1. Build with dependencies ``makepkg -s`` (double check /etc/makepkg.conf for parallel compilation or this will take forever)
1. Install with pacman ``pacman -U qemu-git-<version>.pkg.tar.zst``

# Usage
To toggle hypervisor flags in CPUID:
```qemu-monitor-command --domain win --cmd '{"execute": "toggle-hypervisor"}'```
