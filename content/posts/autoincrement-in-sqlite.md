+++
title = 'SQLite：谨慎使用AUTOINCREMENT'
date = 2024-08-02T19:21:39+08:00
draft = false
+++

## 背景

SQLite默认自带一个名为`rowid`的列，如果没有指定主键，`rowid`会自动成为主键，且自增。

如果指定一个`INTEGER PRIMARY KEY`列，SQLite会自动将其设为`rowid`的别名。此时，`AUTOINCREMENT`是可选的。

## AUTOINCREMENT

`AUTOINCREMENT`存在与否主要影响自增算法。

两种算法的共通点：下一个`rowid`值是当前最大`rowid`值加1。

主要的区别在于：在`rowid`到达最大值（\(2^{63}-1=9223372036854775807\)）时，`AUTOINCREMENT`不会复用已删除/未使用的`rowid`，从而抛出错误；而默认算法会复用。

## 结论

正如官网所言：

> **The AUTOINCREMENT keyword imposes extra CPU, memory, disk space, and disk I/O overhead** and should be avoided if not strictly needed. It is usually not needed.
> 由于`AUTOINCREMENT`会增加额外的开销，应尽量避免使用。
>
> If the AUTOINCREMENT keyword appears after INTEGER PRIMARY KEY, that changes the automatic ROWID assignment algorithm to prevent the reuse of ROWIDs over the lifetime of the database. In other words, **the purpose of AUTOINCREMENT is to prevent the reuse of ROWIDs from previously deleted rows.**
> `AUTOINCREMENT`的目的是防止复用已删除的`rowid`，所以，如果没有这种需求，应尽量避免使用。

## 参考

- [SQLite: AUTOINCREMENT](https://www.sqlite.org/autoinc.html)
