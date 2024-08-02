+++
title = 'Host-only模式下的连通性问题可能是防火墙的锅'
date = 2024-08-01T10:54:43+08:00
draft = false
+++

## 背景

有些VM不方便设置代理，如果没有基于类似OpenWRT的实体设备，新建一个这样的虚拟路由器或许是个不错的选择。

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
![pic_1722578353204](../../assets/pic_1722578353204.png)  
