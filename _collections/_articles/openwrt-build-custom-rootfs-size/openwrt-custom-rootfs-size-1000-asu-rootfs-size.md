---
layout: chapter
chapter_id: 1000
chapter_title: "Upgrade from default firmware to bigger RootFS (Easy way)"
date:   2025-10-18
theme:  openwrt-custom-rootfs-size
permalink: /a/ea0b6ae9-bc03-4dce-a3bb-a2d989083dcc
---

Flash notmal firmware
Install `owut` and `luci-app-attendedsysupgrade`
Set required RootFS size:

```
$ uci set attendedsysupgrade.owut=owut
$ uci set attendedsysupgrade.owut.rootfs_size=512
$ uci commit
```

Set Advanced mode enabled in **System -> Attended Sysupgrade -> Configuration**
Upgrade to the very same version of firmware using **System -> Attended Sysupgrade -> Overview -> Search for firmware upgrade**

That's it.