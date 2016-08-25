---
layout: post
title: Ubuntu upgrade
---

After upgrade from 12.04 to 14.04 i lived for months with old kernel (3.5.x).

The current one (3.13.x) was unable to boot with error "unable to find root device"

Suprisingly everything worked ok, except nvidia hw gfx acceleration.

TIL one need to install linux-image-extras by hand if using hw raid adapter like mpt2sas. In older distro it was part of main package.

After apt-get linux-image-extra, update-grub + update-initramfs and reboot everything works good.