---
layout: post
section-type: post
title: Setting Up VyOS for BGP and OSPF
category: Homelab
tags: [ 'director', 'icinga2', 'guide', 'distributed', 'monitoring' ]
---

## Summary


## Goal

## Steps Taken
![](/img/2020-12-23-setting-up-vyos-9a51f.png)
![](/img/2020-12-23-setting-up-vyos-3f0fe.png)

Install vyos
Go through [quick start guide](https://docs.vyos.io/en/latest/quick-start.html) making sure to setup interfaces with [vlans](https://support.vyos.io/en/kb/articles/vlan-sub-interfaces-802-1q)
![](/img/2020-12-23-setting-up-vyos-53d76.png)
![](/img/2020-12-23-setting-up-vyos-ad1f9.png)
![](/img/2020-12-23-setting-up-vyos-eda83.png)
make sure nat goes out the right eth0.69 interface and does masquerade for each vlan
set protocols ospf area 69 network 10.0.69.0/24
set protocols ospf parameters router-id 10.0.69.1

set interfaces vti vti0 address 192.168.2.249/30
https://docs.vyos.io/en/latest/configuration/vpn/site2site_ipsec.html



## Resources Used
https://support.vyos.io/en/downloads/files/vyos-1-1-8-iso
https://downloads.vyos.io/rolling/current/amd64/vyos-rolling-latest.iso
