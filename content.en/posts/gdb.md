---
title: gdb
description: ""
date: 2023-04-02
tags:
  - 202304
  - gdb
categories:
  - gdb
menu: main
---

## gdb概览

![gdb](/imgs/gdb.png)

x/gx: 表示在以16进制方式查看内存时，以64位为单位（即8个字节）显示地址中的内容。例如，使用命令"x/gx 0x7fffffff"可以查看地址0x7fffffff处的8个字节内容

x/c: 表示在以16进制方式查看内存时，以字符形式显示地址中的内容。例如，使用命令"x/c 0x7fffffff"可以查看地址0x7fffffff处的一个字符内容

参考: <https://github.com/fucking-translation/blog/blob/main/src/lang/rust/14-%E4%BD%BF%E7%94%A8GDB%E8%B0%83%E8%AF%95Rust%E5%BA%94%E7%94%A8.md>