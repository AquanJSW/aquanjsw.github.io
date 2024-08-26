+++
title = '终端并发输出'
date = 2024-08-26T21:23:41+08:00
draft = false
+++

## 背景

如果需要并发运行某个/些命令，且它们有各自的输出，如果把它们放在同一个终端窗口中，输出会混在一起，不易阅读。

把它们分开到不同window/pane中是一个解决方案，具体操作取决于终端。

## Windows Terminal

```powershell
wt -M python main.py `; sp python main.py `; sp python main.py `; mf left `; sp python main.py
```

![Windows Terminal Concurrent Output Example](wt-example.gif)

示例中的`main.py`：

```python
import multiprocessing as mp
import time
import os


def f():
    for i in range(3):
        print(f'[{os.getppid()}-{os.getpid()}] {i}')
        time.sleep(1)


if __name__ == '__main__':
    p = mp.Process(target=f)
    p.start()
    p.join()
    input('Press Enter to exit')
```

## Reference

- [Windows Terminal: 命令行参数](https://learn.microsoft.com/zh-cn/windows/terminal/command-line-arguments?tabs=windows)
