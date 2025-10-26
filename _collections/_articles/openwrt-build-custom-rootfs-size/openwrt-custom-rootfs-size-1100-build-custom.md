---
layout: chapter
chapter_id: 1100
chapter_title: "Build image with needed RootFS size (Hard way)"
date:   2025-10-18
theme:  openwrt-custom-rootfs-size
permalink: /a/334f1e96-80e6-4976-b3c6-1d87aa7f4bb1
---

It may only bee needed if you want to pack packages into your firmware, but you don't have enough space in RootFS


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

# Do not forget

Do not forget to set in UCI:

```
$ uci set attendedsysupgrade.owut=owut
$ uci set attendedsysupgrade.owut.rootfs_size=512
$ uci commit
```