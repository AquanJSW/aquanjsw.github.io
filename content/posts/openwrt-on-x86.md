+++
title = 'Openwrt on X86'
date = 2025-05-29T08:45:27+08:00
draft = false
+++

基本上，按照[OpenWrt官方文档](https://openwrt.org/docs/guide-user/installation/openwrt_x86)的步骤来安装即可。

但是在进行完两步有关扩容的操作后，登陆时可能会遇到以下提示：

```
Failed to execute /usr/libexec/login.sh
```

根据[讨论](https://forum.openwrt.org/t/followed-x86-installation-guide-boot-is-failed/228208)，这是一个因扩容引发的文件系统损坏问题。

重启进入 safe模式后，执行以下命令可以修复：

```bash
fsck.ext4 -y /dev/xxx2 # xxx2 对应ext4分区
```
