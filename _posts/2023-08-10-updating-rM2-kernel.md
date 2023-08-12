---
layout: post
title: Updating the reMarkable 2 Linux Kernel
description: How to manually build and update the rM2 Linux kernel
date: 2023-08-09 00:00:00
hero_image: /img/rM2-boot-log.png
image: /img/rM2-boot-log.png
hero_height: is-large
hero_darken: true
tags: rM2
---

The reMarkable 2 (rM2) is an eInk tablet, based on the i.MX7 SoC. The tablet shipped with a fork of the 4.14 kernel and a custom rootFS built with OpenEmbedded. The vendor kernel is based on the NXP vendor kernel with a large collection of rM2 specific patches on top.

## Company

reMarkable should be congratulated for not locking down the device. It is easy to get root SSH access and they have not locked down the debugging interface. This makes it possible to boot your own boot loader (u-boot) and Linux kernel.

They also use OpenEmbedded to build the rootFS which is great, although it would be much better if they contributed their changes back and released their source code.

 They also release their u-boot and Linux source code, although they do legally have to. Here they could do a better job of upstreaming their changes, this improves the user experience (as it's easier to update kernels) and helps developers. 

## How Linux releases work

A new mainline Linux kernel release happens about every 2-3 months. These releases are the major and minor versions, for example 4.14 and 4.15. Some of these releases are marked as long term support, the 4.14 release is a long term release. Long term releases continue to receive point releases afterwards. At this time of this writing the latest 4.14 point release is 4.14.218.

It is perfectly acceptable to use a long term release (for example 4.14) it is critical that the point releases are regularly updated. These releases fix critical security vulnerabilities as well as other important bugs. To get an idea of how many CVEs (security vulnerabilities) have been fixed for the 4.14 kernel, have a look here: https://www.linuxkernelcves.com/streams/4.14

If you are used to Linux on your desktop you might use long term releases then. For example Ubuntu 20.04 shipped with the 5.4 kernel. Ubuntu 20.04 will continue to use the 5.4 release. The key difference here is that Ubuntu continues to update the point releases of their kernel.

## Remarkable 2

The rM2 shipped with the 4.14.78 kernel. The 4.14 kernel was released in 2017 and the 4.14.78 update was shipped in 2018. The 4.14.78 kernel is unpatched and unsupported, when you purchase the product the software is already out of date.

The rM2 used a kernel that was forked from this branch: https://github.com/Freescale/linux-fslc/tree/4.14-1.0.x-imx. It also seems like there has been no effort to upstream any changes.

reMarkable did ship an update to the 5.4.70 kernel. This kernel was released in October 2020. The 5.4.70 kernel is also unpatched and unsupported, exposing users to security vulnerabilities.

## Mainline Linux

I have been working on supporting mainline Linux on the reMarkable 2. I have given a talk at FOSDEM about this, which you can watch here: https://archive.fosdem.org/2022/schedule/event/mobile_kernel_tablet/

This means you can use the latest and greatest Linux kernel on your rM2. There are still some patches that aren't yet upstream though. You can see those here: https://github.com/alistair23/linux/tree/rM2-mainline

## Build and install the new kernel

First build the kernel and modules

```shell
ARCH=arm CROSS_COMPILE=arm-none-eabi- make imx_v6_v7_defconfig
ARCH=arm CROSS_COMPILE=arm-none-eabi- make -j8
ARCH=arm CROSS_COMPILE=arm-none-eabi- make modules_install INSTALL_MOD_PATH=./tmp
rm -rf tmp/lib/modules/*/build tmp/lib/modules/*/source
```

Then copy the modules to the rM2

```shell
scp -r tmp/lib/modules/* root@172.16.1.179:/lib/modules/
```

Copy the Linux image and device tree

```shell
scp -r arch/arm/boot/zImage  root@172.16.1.179:/home/root/
scp -r arch/arm/boot/dts/imx7d-remarkable2.dtb  root@172.16.1.179:/home/root/
```

Then SSH to the device. Under `/lib/modules/` you should see the origin 5.4 director and a new directory.

You then want to move the `/home/root/imx7d-remarkable2.dtb` file to `/boot/zero-sugar.dtb` and the `/home/root/zImage` to `/boot/`. Make SURE you back the original files first

After that reboot.
