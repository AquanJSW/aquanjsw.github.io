+++
title = 'OSX-KVM: 修复因TSC导致的panic'
date = 2024-09-16T23:27:04+08:00
draft = false
+++

## 背景

环境：

- CPU: AMD Ryzen 7 5800H
- OS: Ubuntu 24.04 LTS with default kernel 6.8.0
- Deployer: [OSX-KVM](https://github.com/kholia/OSX-KVM)

最开始，只有Big Sur可以正常启动（我只测试了Big Sur及更新的OS），其他OS都因为non-monotonic time而引发panic，你可以在[这里](https://github.com/thenickdude/KVM-Opencore/issues/15)找到相关讨论。

简单而言，这个panic主要发生在使用AMD CPU的平台上，Linux内核因为某些原因禁用了TSC，你可以通过这些命令确认：

```shell
# 可用的时钟源
cat /sys/devices/system/clocksource/clocksource0/available_clocksource
# 当前时钟源
cat /sys/devices/system/clocksource/clocksource0/current_clocksource
# 发生panic后
sudo dmesg | grep -i clocksource
```

如果没有`tsc`输出，且你的CPU支持TSC：

```shell
lscpu | grep -i tsc
```

那就会是这种情况。

如果不解决，那么就只能用Big Sur了，或者只用单核单线程启动Sonoma等新版本，这样就不会发生panic (:<

幸运的是，有一条[回复](https://github.com/thenickdude/KVM-Opencore/issues/15#issuecomment-1604049560)提供了一种解决方案。

## 解决方案

[帖子](https://www.reddit.com/r/Amd/comments/uf0zdf/comment/i6tqak0/)中所提到的内核补丁给内核参数`tsc`启动增加了`directsync`选项，可以看作是一种缓解措施。

下一步就是编译内核咯，一些有用的链接：

- [官方的编译内核文档](https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel)：`binary-perarch`可以不用编译。
- [内核补丁](https://git.uplinklabs.net/steven/ec2-packages/src/branch/master/linux-hsw)：我只用到了0009, 0020, 0025三个补丁。

安装内核时可能会遇到冲突错误，可以按照提示卸载对应的包，然后再次安装。

安装成功后再在`/etc/default/grub`中添加`directsync`选项：

```shell
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash tsc=directsync"
```

然后更新grub，就可以重启了。
