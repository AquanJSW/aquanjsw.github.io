+++
title = 'Firewall影响Host-Only模式下连通性一例'
date = 2024-08-01T10:54:43+08:00
draft = false
categories = ['Network']
tags = ['OpenWRT', 'Firewall', 'Virtual Machine']
+++

## 背景

有些VM不方便设置代理，通过一个OpenWRT VM的透明代理进行连接是一个不错的选择。

## 问题

给OpenWRT VM一个Host-only网卡可以用于ssh连接，但host和guest之间无法ping通。

## 解决

在两边都设置防火墙。

**Guest：**

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

**Host (Windows):**

添加一个防火墙的入站规则，只用一个过滤规则：远程设备加入Host-only虚拟交换机的CIDR地址，如`192.168.233.0/24`
![pic_1722578353204](pic_1722578353204.png)  
