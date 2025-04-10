+++
title = 'Archlinux作为路由器'
date = 2025-04-10T22:09:49+08:00
draft = false
+++

步骤参考[Archlinux的官方教程](https://wiki.archlinux.org/title/Router), 此处是一些笔记.

## 开机Link Up指定interface

官方教程中已经知道如何开机连接PPPoE，但interface没有开机Link up故而导致无法生效

借助[`systemd.network`的`ActivationPolicy`](https://man.archlinux.org/man/systemd.network.5.en#%5BLINK%5D_SECTION_OPTIONS), 比如可以在`/etc/systemd/network/99-default.network`:

```systemd
[Match]
Name=*

[Link]
ActivationPolicy=up
```

## `dnsmasq`作为DNS和DHCP服务器

arch自带的`systemd-networkd`内置DHCPv4功能，`systemd-resolved`作为DNS服务器

但是`systemd-networkd`欠缺DHCPv6相关功能，`systemd-resolved`欠缺过滤AAAA功能

所以用`dnsmasq`替代

### NAT66

如果ISP没有下发IPv6前缀，则需要NAT66:

```ini
# /etc/systemd/network/xx-your-lan-interface.network
[Network]
# You can get one using a ULA generator
Address=fd5b:a8aa:eba7::/48
IPMasquerade=both
```

```ini
# /etc/dnsmasq.conf
# Same prefix as above
dhcp-range=fd5b:a8aa:eba7::, ra-stateless, ra-names
```
