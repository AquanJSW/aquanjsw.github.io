+++
title = 'SR-IOV GPU直通 + QEMU/KVM Windows虚拟机 + Sunshine/Moonlight游戏串流'
date = 2025-04-14T15:42:56+08:00
draft = false
+++

最近入手了一台N100小主机作为路由，前面已经对比实验了[Windows作路由]({{% ref "posts/windows-as-router" %}})和[Linux作路由]({{% ref "posts/arch-as-router" %}})的性能，最终选择了Linux作为路由。

之所以要做上述实验，是考虑到用N100仅作为路由是纯纯的浪费性能，为了充分利用N100的性能，决定加入一个小游戏串流功能。

最终结果还是比较令人满意的，在此记录一下配置过程中的一些坑。

## 开启SR-IOV

首先确定BIOS中的VT-d和SR-IOV都已经开启。

按照[i915-sriov-dkms](https://github.com/strongtz/i915-sriov-dkms)的说明安装即可。

但是本人在尝试直接用`i915-sriov-dkms`AUR时似乎有些问题，重启后并没有生效。最后选择了手动安装。

安装完成并重启，确认SR-IOV已经生效：

1. 通过`lspci`找到GPU的PCI地址，假设为`orig_gpu_bsf="00:02.0"`：
2. 虚拟一个GPU：

   ```bash
   echo 1 > "/sys/devices/pci0000:00/0000:$orig_gpu_bsf/sriov_numvfs"
   ```

3. 再次通过`lspci`确认是否有新的PCI地址出现, 假设为`dest_gpu_bsf="00:02.1"`：

之后我们就可以通过`unbind`与`vfio-pci`两步操作来将GPU的PCI地址从原来的驱动中解绑，并绑定到`vfio-pci`驱动上，最后就可以供QEMU/KVM使用了：

```bash
dest_gpu="0000:$dest_gpu_bsf"
dest_gpu_vd="$(cat /sys/bus/pci/devices/$dest_gpu/vendor) $(cat /sys/bus/pci/devices/$dest_gpu/device)"
echo $dest_gpu > /sys/bus/pci/devices/$dest_gpu/driver/unbind
echo $dest_gpu_vd > /sys/bus/pci/drivers/vfio-pci/new_id
```

由于每次重启后都需要手动执行上述操作，所以我们可以将其写成一个脚本，方便后续使用：

新建一个目录作为虚拟机的工作目录，例如`~/kvm-win`，并在该目录下新建一个脚本`srvio-vfio.sh`：

```bash
#!/bin/bash

orig_gpu_bsf="00:02.0"
dest_gpu_bsf="00:02.1"

setup() {
        set -e
        echo 1 > "/sys/devices/pci0000:00/0000:$orig_gpu_bsf/sriov_numvfs"

        dest_gpu="0000:$dest_gpu_bsf"
        dest_gpu_vd="$(cat /sys/bus/pci/devices/$dest_gpu/vendor) $(cat /sys/bus/pci/devices/$dest_gpu/device)"
        echo $dest_gpu > /sys/bus/pci/devices/$dest_gpu/driver/unbind
        echo $dest_gpu_vd > /sys/bus/pci/drivers/vfio-pci/new_id
        set +e
}

check() {
        lspci -ks $dest_gpu_bsf | grep -q "vfio-pci"
        return $?
}

main() {
        check
        [ $? -ne 0 ] && setup
        check
        [ $? -ne 0 ] && echo "Failed to setup SR-IOV" && exit 1
        exit 0
}

main $@
```

这个脚本是幂等的，重复操作不会有任何影响。

## 配置QEMU/KVM

### （可选）配置TAP

如果需要更高性能的网络，可以考虑使用TAP设备。

由于本次的实验机同时作为了路由，所以具体的配置过程可能不具泛用性，对于一般情况（宿主机仅是DHCP客户端），我会在必要的地方作简要说明，但由于未经验证，不保证可行性。

众所周知，仅仅在命令行中配置的网络是无法在重启后生效的，所以需要通过某种方法使其在虚拟机启动前就生效。

QEMU的`-netdev`参数支持两个script参数，分别是`script`和`downscript`，前者在虚拟机启动前执行，后者在虚拟机关闭后执行。这是配置TAP设备的一个途径。

但由于本次实验通过`systemd`配置了路由，为了统一，我决定使用`systemd`来配置TAP设备。

1. 首先确保所有网络设备开机UP：

   ```ini
   # /etc/systemd/network/99-default.network
   [Match]
   Name=*

   [Network]
   ActivationPolicy=up
   ```

2. 新建一个网桥：

   ```ini
   # /etc/systemd/network/10-br-lan.netdev
   [NetDev]
   Name=br-lan
   Kind=Bridge
   ```

   ```ini
   # /etc/systemd/network/10-br-lan.network
   [Match]
   Name=br-lan

   [Network]
   Address=192.168.2.1/24
   Address=fd9c:cd15:eaaa::/60
   IPMasquerade=both
   ```

   > 如果是DHCP客户端，请在此配置为DHCP客户端模式而非静态地址。

3. 新建一个TAP设备并加入网桥：

   ```ini
   # /etc/systemd/network/10-tap0.netdev
   [NetDev]
   Name=tap0
   Kind=tap
   ```

   ```ini
   # /etc/systemd/network/10-tap0.network
   [Match]
   Name=tap0
   
   [Network]
   Bridge=br-lan
   ```

4. 本机LAN口加入网桥（假设为`eth0`）：

   ```ini
   # /etc/systemd/network/10-eth0.network
   [Match]
   Name=eth0

   [Network]
   Bridge=br-lan
   ```

最后确保`systemd-networkd`已经enable：

```bash
systemctl enable systemd-networkd.service
```

重启后确认配置已经生效：

```bash
ip link
```

确认`tap0`和`br-lan`都已经UP，并且`br-lan`的IP地址已经生效：

```bash
ip addr
```

### 文件准备

- `OVMF_VARS*.fd`和`OVMF_CODE*.fd`

  用于UEFI引导
  
  Arch下可以通过`edk2-ovmf`安装，然后用`pacman -Ql edk2-ovmf`查看文件位置
  
  建议将`OVMF_VARS*.fd`复制一份到工作目录下，因为它会在每次启动时被修改
  
- `virtio-win-*.iso`

  用于安装virtio驱动
  
  fedorapeople.org提供的[virtio镜像](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)
  
- Windows安装镜像
  
  性能有限的话建议用10而非11
  
### （可选）在本地安装VNC客户端

由于本次实验机为无头机，安装过程要借助VNC来完成

本次实验使用了`TightVNC`
  
### 创建虚拟机

1. 创建虚拟硬盘

   ```bash
   qemu-img create -f qcow2 win.qcow2 256G
   ```

2. 创建启动脚本`boot.sh`

   ```bash
   #!/bin/bash

   ./sriov-vfio.sh

   qemu-system-x86_64 \
	 -machine q35 \
	 -enable-kvm \
	 -cpu host \
	 -smp 4 \
	 -m 8G \
	 -drive if=pflash,format=raw,readonly=on,file=/usr/share/edk2/x64/OVMF_CODE.4m.fd \
	 -drive if=pflash,format=raw,file=OVMF_VARS.4m.fd \
	 -drive file=win10.iso,media=cdrom \
	 -drive file=virtio-win-0.1.271.iso,media=cdrom \
	 -drive if=virtio,format=qcow2,file=win.qcow2 \
	 -usb \
	 -device usb-tablet \
	 -device vfio-pci,host=00:02.1,x-vga=on,multifunction=on \
	 -nic tap,ifname=tap0,script=no,downscript=no \
	 -display vnc=:0
   ```

   > 按需调整必要参数

### 安装系统

启动脚本，VNC远程连接完成安装，注意选择硬盘时需要先安装virtio驱动，然后才能看到硬盘，`vioscsi`或者`viostor`应该都可以，我用的是`vioscsi`

安装完成后，等待系统更新自动安装显卡驱动。

如果正常的话，就能在任务管理器中看到GPU了。

一些建议的优化：

1. 删除Windows账户密码

   手机端输入密码很折磨

2. 允许RDP的无密码连接

   RDP依旧是稳定的调试方案

   将下面内容保存为一个`*.reg`文件，双击导入即可：

   ```registry
   Windows Registry Editor Version 5.00

   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa]
   "LimitBlankPasswordUse"=dword:00000000
   ```

3. 安装虚拟显示器

   （或许）是串流软件正常使用（硬件加速）的必要条件

   有多种选择，我用[Virtual-Display-Driver](https://github.com/VirtualDrivers/Virtual-Display-Driver)测试成功

   安装时安装所有脚本，在右下角托盘处允许开机启动

## 配置串流

尝试了Steam Link，未成功

Parsec和Sunshine/Moonlight都可以成功，但考虑到我在手机端的体验，最终采用了可以预定义应用的Sunshine/Moonlight，因为在手机端用鼠标点来点去启动应用实在是太麻烦了。

Sunshine并不需要多少额外的配置，预设定一下自己想要的程序即可

如果用Moonlight连接时提示什么无法打开应用的话，则需要先连接桌面，登录进入，再返回打开应用试试。

## 其他

- 或许你想要配置一下虚拟显示器：同时打开VNC与Moonlight的桌面模式，这就是Windows虚拟机可以识别到的两个显示器了，VNC的显示器无法使用硬件加速，所以在Windows设置中把Moonlight看到的虚拟显示器设置为主显示器。

  然后再看喜好在Windows设置中调整一下Moonlight看到的虚拟显示器的分辨率
