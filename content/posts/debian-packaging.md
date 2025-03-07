+++
title = 'Debian Packaging'
date = 2025-03-05T09:35:22+08:00
draft = false
categories = ['Linux']
tags = ['Debian Packaging']
+++

## 结论

由于Debian Package Repository本质上只是一个HTTP服务器, 并不限于Launchpad PPA, 只要满足一定的文件结构[^format], 我们就可以使用GitHub Pages[^hosting]或自建服务器来托管Debian Package Repository.

虽然Launchpad PPA是主流选择, 但由于其上传流程比较繁 (上传后不能查看实时构建状态, 需要等待邮件通知, 而且等待时间较长), 所以除非必要, 我们可以借助Github Pages或自建服务器来托管Debian Package Repository.

另外, 本次Debian Packaging源于对github项目的Publish渠道尝试, 除非有特殊需求, 可以使用Github自带的Packages功能发布Container, 这样也可以在用户侧用一条命令安装我们的软件, 以无法自动更新为代价.

然而, 学会Debian Packaging还是些帮助的, 如果有systemd服务文件, deb包会自动安装, 在github Release中发布deb包, 也可以方便用户部署.

## 实践记录

> 注意:
>
> - 本文并不保证流程的合理性, 但会保证流程的可行性.
> - 完整的Debian Packaging细节可以参考手册[^manual], 本文只作为作者的实践记录.
> - 本文是关于组织Source Debian Package的流程记录, 该流程通过编译源码生成deb包, 而非直接通过二进制文件组织deb包.
> - 在debian官方gitlab[^salsa]上有很多项目的源码, 可以参考.

```bash
sudo apt update
sudo apt install build-essential dh-make devscripts git
# 创建一个工作目录, 然后在工作目录下clone源码到本地 
# (这是因为打包过程中会在源码的上层目录生成一些文件)
mkdir -p $workspace && cd $workspace
git clone $url && cd $project_name
# 切换到想要构建的目标版本tag, 然后根据该tag起一个合适的debian版本号version
# 例如: tag=v1.0.0, version=1.0.0
git checkout $tag
dh_make -p $package_name-$version --createorig
```
> 注意, 实际中debian版本号可能会有很多部分, 例如[^uploading][^manual]:
> `myapp_1.0-2~ppa1~ubuntu17.04.1`
> - `myapp`: 包名
> - `1.0`: 上游版本号, 上游发布新版本时, 更新这个部分
> - `2`: debian修订号, 一般情况下, 都会有这个部分, 从1开始, 每次为了适配debian打包而对上游打补丁时, 更新这个部分
> - `ppa1`: ppa修订号, 每次在ppa发布新版本时, 更新这个部分
> - `ubuntu17.04.1`: 该包是针对ubuntu17.04发布的, 修订号为1, 在目标系统发布新版本时, 更新这个部分
> 
> 以上这种版本号会需要在`debian/changelog`中体现, 但此处, 简单起见, 我们只用上游版本号即可, 后续可以随时修改


经过上述`dh_make`后, 会在上层目录生成一个`orig.tar.*`文件, 该文件可以保证源码的完整性

初始构建我们可以直接利用`dh_make`从git源码生成的`orig.tar.*`文件, 后续构建可以使用`uscan`命令从上游获取最新源码, 这在后面会有介绍

当然, 如果你本人就是项目的维护者, 那么你总是可以使用类似于这种的方法从git源码生成`orig.tar.*`文件

接下来, 需要将debian目录下的文件进行修改, 以适应我们的项目, 详细信息请参考手册[^manual]:

- `debian/control`: 
    - 各种Depends字段中, **如果包含了特定系统的依赖, 并不需要在版本号中体现**, 因为这些依赖是针对构建时的系统的, 而不是针对目标系统的. 如果想要对构建时系统有所要求, 可以在`debian/rules`中写入检测系统的逻辑, 比如:
        ```makefile
        distro_codename = $(shell lsb_release -sc)
        ifneq ($(distro_codename),noble)
            $(error This package can only be built on Ubuntu 24.04)
            exit 1
        endif
        ```
    - 为了满足Source字段要求, 如果上游项目名称不符合要求 (比如包含大写字母), 那么需要修改
- `debian/changelog`:
    - 如果是针对特定系统的包, 除了需要在版本号中体现, 还要将distribution字段改为对应的系统名, 如`focal`, `bionic`等
    - 对于使用`dh_make --createorig`生成过orig.tar.*文件的初次构建情况, 注意上游版本号的一致性
- `debian/copyright`
- `debian/rules`: 
    - 该文件可以用来控制构建过程, 拜`dh`系列工具所赐, 即便使用由`dh_make`生成的默认文件, `debuild` (或 `dpkg-buildpackage`) 命令也可以正常运行 (即便并不会生成我们想要的deb包) 
    - 之所以使用`dh`系列工具, 因为它是现在的主流 (`dh_make`默认生成的`debian/control`文件中已经加入了对该系列工具的依赖, 比如`debhelper-compat (= 13)`). 工具列表以及各个工具的行为请见manpage[^dh7], 系列中的大多数工具的默认行为就是我们想要的, 但如果有特殊需求, 可以在这里进行修改, 只需要在`debian/rules`文件中添加对应的命令的`override_*`目标即可.
    - 特别地, 我们关注一下`dh_auto_build`, `dh_install`两个工具, 这两个工具对生成deb包尤为重要
        - `dh_auto_build`
            - 默认该工具会在项目目录下执行`make`命令 (也有其他行为, 相见对应的manpage), 如果我们的项目中正好有可供`dh_auto_build`默认使用的构建文件, 那么我们就不需要在`debian/rules`中添加`override_dh_auto_build`目标, 否则, 我们需要在该目标中添加构建命令
        - `dh_install`
            - 该命令负责将需要打包入deb的文件安装到临时目录, 该临时目录后续会被直接用来生成deb包. 其实`dh_install`有一系列姊妹工具, 比如`dh_installman`, `dh_installinit`等, 建议优先用这些姊妹工具, 而不是直接使用`dh_install`, 即我们要将`dh_install`当作fallback安装工具.
            - 一般来说, 我们并不需要在`debian/rules`中添加`override_dh_install`目标, `dh_install`及其姊妹工具会自动检索`debian`目录下的文件用于确认安装源和目标位置, 比如`dh_install`就会查找`debian/<package_name>.install`文件. 有关这些安装文件的格式, 请参考对应的manpage, 此处只放一行示例:
                ```plaintext
                output/${env:ARCH}/release/* usr/lib/
                ```
                > 第一项是源文件路径 (此处使用了通配符和变量替换功能. 该路径是项目目录下的相对路径)
                > 第二项是目标文件路径, 我们并不需要了解目标文件路径的绝对位置 (如果你想知道, 可以看对应的manpage), 这只是一个临时目录

以下是一些常用的可选文件:

- `debian/<package_name>.{preinst,postinst,prerm,postrm}`:
    - 如果需要在安装/卸载时执行一些脚本, 可以在这里添加, 比如创建链接, 创建用户等. 详情请参考手册[^manual]
- `debian/<package_name>.service`:
    - 隶属于`dh_installsystemd`工具, 用于安装systemd服务文件
- `debian/source/options`:
    - 如果在编译过程中会有一些有关`dpkg-source`警告, 可以在这里添加忽略规则, 比如可以用来忽略一些构建过程中产生的临时文件, 否则会在`debuild`时报错:
        ```plaintext
        extend-diff-ignore = "node_modules|dist"
        ```
    > 如果想要忽略构建过程中其他类型的警告, 请从对应的manpage中寻找如何忽略, 一般会放在`debian/source/`目录下, 例如`debian/source/lintian-overrides`用于忽略lintian警告

最后, 构建命令可以使用`debuild -us -uc`或`dpkg-buildpackage -us -uc` (这两个命令的区别在于`debuild`会自动安装构建依赖, 而`dpkg-buildpackage`不会), 构建成功后, 会在上层目录生成deb包

如果想要上传到Launchpad PPA或组建自己的Debian Package Repository, 可以参考这篇文章[^hosting]

[^hosting]:[Hosting your own PPA repository on GitHub](https://assafmo.github.io/2019/05/02/ppa-repo-hosted-on-github.html)
[^format]:[Debian Repository Format](https://wiki.debian.org/DebianRepository/Format?action=show&redirect=RepositoryFormat)
[^uploading]:[Packaging/PPA/Uploading](https://help.launchpad.net/Packaging/PPA/Uploading)
[^manual]:[Debian Policy Manual](https://www.debian.org/doc/debian-policy)
[^dh7]:[debhelper(7) manpage](https://man7.org/linux/man-pages/man7/debhelper.7.html)
[^salsa]:[Debian GitLab](https://salsa.debian.org/public)