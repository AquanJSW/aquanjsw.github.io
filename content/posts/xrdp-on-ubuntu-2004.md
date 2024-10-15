+++
title = 'xrdp on Ubuntu 20.04'
date = 2024-10-15T14:26:51+08:00
draft = false
categories = ['Linux']
tags = ['xrdp', 'Ubuntu', 'gnome']
+++

手里有个NVIDIA AGX Xavier, 这个型号似乎也就停止在Ubuntu 20.04了，想要使用更新版本
Ubuntu中的新版高性能grd也就不可能了，所以只能使用xrdp了。

主要记录几个问题的解决过程:

- （可能并无必要）把这个问题询问了Github Copilot，提示xrdp可能想要占用6010端口，
  需要关闭sshd的X11转发
- NV给的`$HOME/.xsessionrc`中的第83行`remove_app=...`似乎会导致错误，这点可在
  `$HOME/.xsession-errors`中查看，注释掉即可
- 如果想要使用`gnome`，需要在`$HOME/.xsession`中添加：
  
  ```bash
  export DESKTOP_SESSION=ubuntu
  export GNOME_SHELL_SESSION_MODE=ubuntu
  export XDG_CURRENT_DESKTOP=ubuntu:GNOME

  gnome-shell --replace
  ```
