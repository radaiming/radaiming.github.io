---
layout: post
title: "cross dev uder Debian"
categories: linux
---

Recently I again have some mipsel cross dev work assigned, after struggling for a while, I found that Debian is very friendly to cross dev, and I'm using two ways to do cross dev.

### cross dev with Debian's multiarch support
Debian supports installing softwares of other architecture with multiarch:

~~~~~~~~
# dpkg --add-architecture mipsel
# apt-get update
# apt-get install libc6:mipsel
~~~~~~~~
After installing gcc, we could start compilation. There're already plenty pre-compiled packages, so usually there would not be much trouble on package dependency.

### cross dev with QEMU and chroot
This is more reliable than the above way. First we could use [debootstrap](https://wiki.debian.org/Debootstrap) to bootstrap a mips root filesystem, or simply use gentoo's stage3, then do as Debian's [wiki](https://wiki.debian.org/QemuUserEmulation) said. Compilation could be very slow, but at least it works.
