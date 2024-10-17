+++
title = '选择 Python 虚拟环境工具'
date = 2024-10-16T18:27:58+08:00
draft = false
categories = ['Utility']
tags = ['Python']
+++

Python 虚拟环境的必要性不必多说，创建虚拟环境的工具有很多，且都大同小异，
所以选一个自己喜欢的就好。

由于我偶尔有对 postactivate 这类脚本的需求，virutalenvwrapper 和 conda
可以满足条件。

- 对于 Linux 平台，两者可以互补使用。一般情况下可以使用 conda ，
对于需要引用 *system-site-packages* 的情况，可使用 virtualenvwrapper

- 对于 Windows 平台，一方面没有 *system-site-packages* 的问题，
另一方面 virtualenvwrapper 并没有原生支持，所以就选 conda 咯
