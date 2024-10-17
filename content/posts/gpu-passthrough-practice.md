+++
title = '记录一次GPU Passthrough的实践'
date = 2024-10-14T14:14:04+08:00
draft = false
categories = ['Linux']
tags = ['IOMMU', 'QEMU/KVM', 'GPU Passthrough', 'libvirt']
+++

教程基本按照

- [CSDN：Ubuntu双显卡GPU Passthrough](https://blog.csdn.net/ahmclishihao/article/details/132679686)
- [ArchWiki: PCI passthrough via OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

过程中发现了一些问题，有些也无法解决：

- 由于Host有双网卡，所以想要直通一块给Guest，即加入了内核参数
  `intel_iommu=on vfio-pci.ids=8086:15b7`，不过重启后Host仍然占用了这块网卡。
  好在virt-manager中仍然可以添加这块网卡并正常使用。然而当Guest多次重启后，查看
  `dmesg`会发现有`vfio-pci`的错误，并导致网卡直通失败，此时只能重启Host才能解决。
- 根据[ArchWiki：CPU pinning](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#CPU_pinning),
  为了避免CPU频繁切换，可以将Host的CPU核心固定，但这似乎只对部分CPU生效，
  因为我手中的Intel的所有CPU共享一个L3缓存，所以无法固定。
  实际进行游戏性能测试时也发现性能确实存在问题，或许与此相关。
- 对于SATA SSD，由于不走PCI通道，所以无法直通，只能虚拟化。
  但不论是虚拟化还是半虚拟化（virtio），虽然在Guest里测试了读写速率似乎没有影响，
  但Guest确实识别为了HDD，而不是SSD，这可能会影响性能。
- 对于Windows Guest，SATA SSD改为virtio模型的最好方法是：先创建一个virtio的小虚拟硬盘，
  启动Guest，安装virtio驱动，当可以识别到小硬盘时，再将原来的SATA硬盘改为virtio模型。
- Windows Guest的登录问题其实很恼人。对远程桌面使用来说，有两种：
  - RDP：这种方法无需连接显示器，适合办公，但不适合游戏。
  - Parsec, Sunshine/Moonlight：这种方法需要连接显示器，适合游戏。
    但问题是，如果连有显示器（无论是直接连接屏幕、Dummy Plug还是虚拟显示器），
    那么就会面对双显示器一个在里面（例如virt-manager的模拟显示器），一个在外面的情况，
    主显示器的设置就很让人头疼，无论设置为哪一个，都会有问题。
    如果可以设置为镜像显示就还好，但目前我并没有发现qxl这种模拟显示器支持镜像显示，
    因为Windows Guest的设置里压根就没有镜像的选择。

最后也是直接卸载了，对我来说并无什么实际用途，只是很早之前就想尝试一下 :)

最后有一个小插曲，在去掉GPU的直通脚本和相关的内核参数、重新生成initramfs并重启后，
虽然正确识别到了两张显卡，但gdm3挂了，卸载libvirt后恢复正常。
