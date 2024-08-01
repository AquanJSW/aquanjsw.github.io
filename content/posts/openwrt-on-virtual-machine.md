+++
title = 'OpenWRT on Virtual Machine'
date = 2024-08-01T10:54:43+08:00
draft = false
+++

## Background

As a virtual router with a transparent proxy for VMs that are not convenient to set up a proxy.

## Notes

### Try to set up firewalls on both sides if network problems occur

My scenario involves a connectivity issue where there is an inability to ping each other through host-only virtual switch between a Windows host and an OpenWRT client.

Solution is to set up a firewall on both sides.

**Client side:**

```plain
# /etc/config/network
...
config interface 'hostonly'
        option device 'eth0'
        option proto 'static'
        option ipaddr '192.168.233.2'
        option netmask '255.255.255.0'
        option ip6assign '60'
...
```

```plain
# /etc/config/firewall
...
config zone
        option name             lan
        list   network          'hostonly'
        option input            ACCEPT
        option output           ACCEPT
        option forward          ACCEPT
...
```

**Host side (Windows):**

Add a firewall inbound rule with remote address filtering set to the virtual switch's CIDR like `192.168.233.0/24`
