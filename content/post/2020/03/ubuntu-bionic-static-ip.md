---
title: 给ubuntu Bionic设置静态IP
date: 2019-12-22 23:24:36
tags: ["linux"]
---

vim /etc/netplan/01-netcfg.yaml

<!--more-->
```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      addresses: [192.168.1.22/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [192.168.1.1]
        ```