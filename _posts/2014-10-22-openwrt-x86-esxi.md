---
published: true
layout: post
title: OpenWRT x86 in VMWare ESXi
---
In ESXi routing, NAT and other advanced networking must be accomplished through Router Virtual Machine. DD-WRT is fine example which works stable, but it costs money to run it on x86.

You can use OpenWRT the same way as DD-WRT.
To create image yourself, download [image](https://downloads.openwrt.org/barrier_breaker/14.07/x86/generic/) and convert it using qemu-img.
I`ve used openwrt-x86-generic-combined-ext4.img.gz.

```
qemu-img convert -f raw -O vmdk openwrt.img openwrt.vmdk
```

After that create ESXi other Linux 3.x 32 bit machine (without CD-ROM, Printer and etc). Remove SCSI disk and add IDE and upload created VMDK file.

Make sure to set Ethernet adapter to E1000.

![OpenWRT screenshot](http://i.imgur.com/Ne2BTPC.png)
