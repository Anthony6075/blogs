---
title: Linux加载器、加载、动态链接器与动态链接
date: 2024-05-05 15:35:36
categories:
- computer_science
- linker
tags:
- linux
- loader
- linker
---

# 1. 别名和对应的文件

别名包括:

动态链接器(dynamic linker)，加载器(loader)，动态加载器(dynamic loader)，运行时链接器(run-time linker)，ELF解释器(ELF interpreter)，ld.so，ld-linux.so

上述这些描述的是同一件事，其对应的文件为:

ld.so，ld-linux.so，/lib/ld-linux.so.1，/lib/ld-linux.so.2，/lib64/ld-linux-x86-64.so.2

在现代linux机器上，一般使用的是/lib/ld-linux.so.2和/lib64/ld-linux-x86-64.so.2，用于处理ELF格式二进制文件

# 2. 动态链接器运行方式

1. 间接运行: 通过运行一个动态链接的程序或者共享库，ELF文件会将动态链接器存放在.interp段中

2. 直接运行: /lib/ld-linux.so.*  [OPTIONS] [PROGRAM [ARGUMENTS]]

# 3. 动态链接器作用

三步，(1)解析并加载程序所需的共享库依赖，(2)准备运行程序，(3)运行程序

# 4. 动态链接器解析动态库依赖的过程

1. 检查每个依赖名(dependency string)是否包含字符`/`

    1.1 如果包含，将此依赖名当做相对路径名或者绝对路径名使用

    1.2 如果不包含，转入第2步

2. 依次在以下位置寻找依赖

    2.1 DT_RPATH(deprecated)

    2.2 环境变量**LD_LIBRARY_PATH**指定的目录，此变量中以冒号或者分号分隔目录，空目录名表示当前工作目录

    2.3 二进制文件的DT_RUNPATH属性

    2.4 /etc/ld.so.cache

    2.5 /lib，/usr/lib，/lib64，/usr/lib64 (还可能包括/lib/x86_64-linux-gnu)