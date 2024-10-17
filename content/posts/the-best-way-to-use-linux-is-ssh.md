+++
title = '使用 Linux 的最佳姿势是 SSH'
date = 2024-10-17T10:55:26+08:00
draft = false
categories = ['Linux']
tags = ['SSH', 'Wayland', 'X11', 'Gnome']
+++

从 Ubuntu24.04 LTS 仍然默认使用 X11 就知道 Linux 下的 DE 发展有多混乱，换成
Wayland 也确实发现仍有 BUG，比如 Chrome 的花屏问题 :(

如果有想要使用 GUI 的需求，可以考虑远程桌面，我测试的目前性能最好的是 gnome-remote-desktop，
这需要较新的 Gnome 版本，我在 Ubuntu 24.04 上测试可以流畅使用，
[教程](https://www.202016.xyz/2024/09/02/gnome-wayland-remote-desktop.html)

可以买个 Dummy Plug 来硬件模拟显示器，省掉了软件模拟的麻烦 :)
