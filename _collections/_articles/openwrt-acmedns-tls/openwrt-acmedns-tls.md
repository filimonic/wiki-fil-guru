---
layout: theme
title:  "TLS Certificates for internal OpenWrt devices using ACME-DNS.IO, no servers, no real IPs, no DNS server API"
date:   2025-05-08
theme:  openwrt-acmedns-tls
permalink: /t/openwrt-acmedns-tls
---

ACME DNS is public service to fullfill DNS-01 challenges.

You MUST understand that you're delegating TLS certificate management to third-party ACME DNS service.

## DNS Names for internal devices

First, create fully qualified DNS names for your OpenWRT devices. In my case, I have access point running OpenWRT 24.10.1:

```txt
ap1.pluto.freipa.xyz 172.19.22.241
ap2.pluto.freipa.xyz 172.19.22.242
```

## Register to ACME-DNS.IO

On Windows, run:

```powershell
irm -Method Post https://auth.acme-dns.io/register
```

You will get something like:

```powershell
username   : 7a8a518d-55c1-494f-a38f-763b6bca1ed3
password   : zaaaaaaab89tN-yqqEocufZDWgfVxg5EErkxKNlz
fulldomain : 200513c6-e899-4916-8380-1b4c5020d540.auth.acme-dns.io
subdomain  : 200513c6-e899-4916-8380-1b4c5020d540
allowfrom  : {}
```

## Register public DNS record

Get `fulldomain` from response above and register `CNAME` record with your DNS provider:

```dns
_acme-challenge.ap1.pluto.freipa.xyz   CNAME   200513c6-e899-4916-8380-1b4c5020d540.auth.acme-dns.io
_acme-challenge.ap2.pluto.freipa.xyz   CNAME   200513c6-e899-4916-8380-1b4c5020d540.auth.acme-dns.io
```

## Enable HTTPS access

On both devices, I enabled HTTPS access (System -> Administration -> HTTP(S) Access -> Redirect to HTTPS)

## Install ACME client

* Install `luci-app-acme` package. It will install dependencies.
* install `acme-acmesh-dnsapi` package.
* Reboot device after install.

## Configure ACME client

* Go to **Services** -> **ACME Certificates** and add new entry with name `device_certificate`.
* Change **Account email** to your email
* On **General settings** tab, tick **Enabled**, set **Domain names** to FQDN of your AP (`ap1.pluto.freipa.xyz`) and select **Validation method** as **DNS**.
* On **DNS Challenge validation** tab, select **ACME DNS API github.com/joohoi/acme-dns**
  * Set **ACMEDNS URL** to `https://auth.acme-dns.io`
  * Set **ACMEDNS User** to *username* value from registration request (`7a8a518d-55c1-494f-a38f-763b6bca1ed3` in example)
  * Set **ACMEDNS Password** to *password* value from registration request (`zaaaaaaab89tN-yqqEocufZDWgfVxg5EErkxKNlz` in example)
  * Set **ACMEDNS Subdomain** to *subdomain* value from registration request (`200513c6-e899-4916-8380-1b4c5020d540` in example)
* On Advanced settings, set **Key type** to **ECC 256 bit**
* Click **SAVE**
* Delete `example_*` certificate configs
* Press **Save and apply**
* Reboot device

## Issue certificates

Wait until midnight or logon to router using ssh and execute:

```shell
/etc/init.d/acme start
```

## Set up LuCI to use issued certificates

* Logon to router using ssh.
* Ensure certificates issued. Run `ls -R /etc/ssl/acme/*.key /etc/ssl/acme/*.fullchain.crt` and result will be like this

  ```shell
  root@ap1:~# ls -R /etc/ssl/acme/*.key /etc/ssl/acme/*.fullchain.crt
  /etc/ssl/acme/ap1.pluto.freeipa.xyz.fullchain.crt
  /etc/ssl/acme/ap1.pluto.freeipa.xyz.key
  ```

* Set **uhttpd** params to use those certificates through **uCI** and reboot:

  ```shell
  uci set uhttpd.main.cert=/etc/ssl/acme/ap1.pluto.freeipa.xyz.fullchain.crt
  uci set uhttpd.main.key=/etc/ssl/acme/ap1.pluto.freeipa.xyz.key
  uci commit
  reboot
  ```

## Check Web UI in new browser tab

Open your device page using FQDN name in a new browser tab.

## Repeat for second AP

Repeat for second ap (`ap2.pluto.freeipa.xyz`).
