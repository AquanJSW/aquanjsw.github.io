+++
title = 'Windows作为路由器'
date = 2025-04-11T21:40:01+08:00
draft = false
+++

其实是用Windows下的一个openwrt虚拟机作为路由器

但是实测性能似乎有问题，用`iperf3`测试，速度很不稳定且跑不满局域网带宽

```plain
Connecting to host 192.168.2.193, port 5201
[  5] local 192.168.2.161 port 7860 connected to 192.168.2.193 port 5201
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.01   sec  21.5 MBytes   178 Mbits/sec
[  5]   1.01-2.01   sec  26.0 MBytes   218 Mbits/sec
[  5]   2.01-3.01   sec  40.6 MBytes   342 Mbits/sec
[  5]   3.01-4.01   sec  47.2 MBytes   398 Mbits/sec
[  5]   4.01-5.01   sec  12.5 MBytes   104 Mbits/sec
[  5]   5.01-6.00   sec  28.0 MBytes   236 Mbits/sec
[  5]   6.00-7.01   sec  11.8 MBytes  98.3 Mbits/sec
[  5]   7.01-8.01   sec  28.8 MBytes   240 Mbits/sec
[  5]   8.01-9.01   sec  40.0 MBytes   336 Mbits/sec
[  5]   9.01-10.01  sec  47.6 MBytes   401 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-10.01  sec   304 MBytes   255 Mbits/sec                  sender
[  5]   0.00-10.01  sec   304 MBytes   255 Mbits/sec                  receiver
```

仅从表面看，CPU N100占用已经100%

综上，或许不太适合Windows + OpenWRT on VMWare作为路由方案

笔记：

## 网络结构

OpenWRT拥有几个虚拟网卡：

- 一个**Host-only**: 在VMWare的虚拟网络设置中禁用该网卡的DHCP功能, 在OpenWRT内将对应interface纳入`br-lan`内. 由于VMWare会在宿主机建立一个虚拟网卡, 再在Windows宿主机的网络面板中设置该网卡为自动获取IP. 这样一来, 宿主机就成为OpenWRT的一个客户端了
- 两个**Bridge**: 在VMWare的虚拟网卡设置中分别桥接宿主机的物理接口(如果有更多接口，就建立对应数量的桥接网卡), 在OpenWRT中根据需求将一些网卡加入`br-lan`中作为LAN口，将连接外网的网卡作为WAN口进行配置. 在宿主机的网络面板上, 取消勾选那些作为LAN的网卡的IPv4和IPv6协议栈
