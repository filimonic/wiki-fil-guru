---
layout: chapter
chapter_id: 1170
chapter_title: "Extra: Forward to AD NPS (unfinished)"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/d57f643f-8cc3-4957-bf99-050ae97e5982
---


## Create files

Create home servers file for wifi: 

```
touch /etc/freeradius/3.0/sites-available/wifi.home_servers
ln --symbolic --target-directory=/etc/freeradius/3.0/sites-enabled ../sites-available/wifi.home_servers
chown -R freerad:freerad /etc/freeradius/3.0/sites-*
chmod 640 /etc/freeradius/3.0/sites-available/*
```

# Edit files

As always, much more details are in `proxy.conf` in FreeRADIUS installation.

Add home servers, home server pool and realm:
I will dedine two `home_server`s, but I will use only first.

```config
# /etc/freeradius/3.0/sites-available/wifi.home_servers

home_server wifi@ad-nps-1 {
    type = auth
    ipaddr = 172.19.21.14
    port = 1812
    secret = Testing123
    response_window = 20
    response_timeouts = 5
    zombie_period = 40
    status_check = status-server
    num_answers_to_alive = 3

}

home_server wifi@ad-nps-2 {
    type = auth
    ipaddr = 172.19.21.15
    port = 1812
    secret = Testing123
}

home_server_pool wifi@ad-nps-pool {
  # Multiple home_server may be defined.
  # Please note that type of switching should NOT be `load-balance` as 
  #   EAP does not work good with. 
  # Use `client-balance` or `client-port-balance` for EAP balancing
  # Or `fail-over` for no balancing at all  
  type=client-port-balance

  home_server = wifi@ad-nps-1
  # home_server = wifi@ad-nps-2
}

realm wifi@ad-nps-realm {
    auth_pool = wifi@ad-nps-pool
    nostrip
}

```

## Define the rule we pass RADIUS request to proxy

I defined anonymous identity for EAP in [client config](/a/87dd6cec-f296-41f5-b488-8bdbb73b2aab)
as `anonymous-od-type-a`. This is what the RADIUS server sees in first RADIUS packet. 

Based on this *anonymous* identity, I will proxy Access-Request to the home server: 

Modify `if (&User-Name !~ /^anonymous-od-/i) {` block in `authorize` section 
in `/etc/freeradius/3.0/sites-available/wifi` configuration to this: 

```
server wifi {
# ...
  authorize {
    # ....
      # Check if user-name starts with `anonymous-od-`
      # If not, I proxy request to AD NPS home server
      # `i` means case-insensitive
      if (&User-Name !~ /^anonymous-od-/i) {
        update control {
          &Proxy-To-Realm := 'wifi@ad-nps-realm'
        }
        return
      }
```