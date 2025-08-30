---
title: Linux ldd命令使用方法
date: 2024-05-05 15:36:13
categories:
- computer_science
- linker
tags:
- linux
- shell
- loader
- linker
---

# 1. 作用

打印可执行程序或者动态库的动态库依赖

# 2. 工作原理

ldd是一个可执行shell脚本，其设置LD_TRACE_LOADED_OBJECTS环境变量为非空值，调用动态链接器ld.so

例如，为了查看ls命令的依赖，`ldd /usr/bin/ls`

相当于

`LD_TRACE_LOADED_OBJECTS=1 /lib64/ld-linux-x86-64.so.2 /usr/bin/ls`