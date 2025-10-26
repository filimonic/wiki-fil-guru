---
layout: theme
title:  "OpenWrt: Using release image with enlarged RootFS (not ExtRoot)"
date:   2025-10-18
theme:  openwrt-custom-rootfs-size
permalink: /t/openwrt-custom-rootfs-size
---
If you have router with extended flash size (ex, NanoPI R5C with 32GB EMMC), you may find that default image has very small RootFS (about 100 MiB). It's a good Idea to extend it, but when you upgrade, you will loose your data in it. 

# The way I do this: 
* first time I flash the router with OpenWrt, I use SquashFS image (like most routers do), but with extended RootFS size 512 MiB. 
* Also I add `owut` and `luci-attended-sysupgrade` packages at first.
* When I need upgrade fw, with theese tools I just request new image with extended RootFS and set of packages I installed.

## Why not ExtRoot?

* ExtRoot script uses all your EMMC, while I think that only reasonable space should be used for the system.
* When you upgrade, ExtRoot may be deleted, so you need to keep it in mind

## Why not to use all EMMC for root?

* You should use reasonable space for system. Exta space can be taken by other partitions for data, but this should be separate partition
* Attended SysUpgrade servers are limited to build 1Gb RootFS

## Why 512 MiB and not 1024 MiB?

* Hitting limits is never good. Leave some room to grow in emergency case
* If 512 MiB is not enough for router, then you probably doing something wrong, like sroting data on RootFS. Create a separate partition for it.

## There are two ways to have a firmware with custom RootFS size: 

* Build firmware with big RootFS locally (Hard way)
* Install normal Firmware, set desired RootFS size in options and then upgrade it using ASU (automated sysupgrade) (Easy way)