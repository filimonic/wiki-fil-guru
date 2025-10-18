---
layout: theme
title:  "OpenWrt: Building default release image with enlarged RootFS (not ExtRoot)"
date:   2025-10-18
theme:  openwrt-build-custom-rootfs-size
permalink: /t/openwrt-build-custom-rootfs-size
---

If you have router with extended flash size (ex, NanoPI R5C with 32GB EMMC), you may find that default image has very small RootFS (about 100 MiB). It's a good Idea to extend it, but when you upgrade, you will loose your data in it. 

# The way I do this: 
* first time I flash the router, I use SquashFS image (like most routers do), but I build it with extended RootFS size 512 MiB. 
* Also I add `owut` and `luci-attended-sysupgrade` packages at build time.
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


# Building

I did not find any Online way to create this easy as [Firware Selector](https://firmware-selector.openwrt.org/) does, that is I'm going to do it offline.

I need:

* Internet access.
* Docker installed. For Windows, Docker Desktop may be used.
* Platform name, Profile name, default package list, openWrt version
  * Go to [OpenWrt ToH](https://toh.openwrt.org), find your device and click it. 
  * Click `Firmware selector` page and note the URL, ex `https://firmware-selector.openwrt.org/?target=rockchip/armv8&id=friendlyarm_nanopi-r5c`
  * `Target` is created from `target` URL part, replacing `/` witn `-`, like `rockchip-armv8` (without `&` symbol)
  * `Profile` is created from `id`, taken as-is, like `friendlyarm_nanopi-r5c`.
  * `Default package list` is taken as-is from expandable `> change packages .. ` option
  * `OpenWrt version` is taken from this page, take latest, not SNAPSHOT.
* Download repo `https://github.com/filimonic/openwrt-r3s-builder` from here and unzip if needed

I build:

* Go to downloaded folder and create a copy of `docker-compose.yml`, name it as you want
* Open for editing, 
* replace args 
  add `   owut luci-app-attendedsysupgrade` to the end of packages.
* save
* open command line and go to this folder and run 
* You need to start Docker (on windows, just start Docker Desktop app)
* run `docker compose --file "your-file-name.yaml" run --build main`

You will find newly created `out` folder with built firmware