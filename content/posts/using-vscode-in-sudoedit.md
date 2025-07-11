+++
title = '用VSCode编辑需要root权限的文件'
date = 2024-08-09T13:58:03+08:00
draft = false
categories = ['Utility']
tags = ['VSCode']
+++

命令行下，可以用`sudo -e`或`sudoedit`命令编辑需要root权限的文件。

---

根据`man sudo`：

> The editor specified by the policy is run to edit the temporary files.
> The sudoers policy uses the SUDO_EDITOR, VISUAL and EDITOR environment
> variables (in that order).

所以可以在`.bashrc`中如此设置：

```bash
export EDITOR='vim' # 默认编辑器
export VISUAL="$(which code) -w"
```

这样，在VSCode的集成终端中，可以直接用`sudoedit`命令启动VSCode编辑需要root权限的文件。
