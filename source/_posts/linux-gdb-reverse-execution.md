---
title: Linux GDB 回退执行
categories:
- computer_science
- gdb
tags:
- linux
- gdb
- shell
---

# 1. 回退执行

使用命令`record`，`reverse-next`，`reverse-step`，`reverse-continue`，`reverse-finish`等等

但在支持AVX(Advanced Vector Extension)的现代CPU上`record`命令不能使用，因此GDB回退执行的功能很鸡肋

glibc的string相关函数会使用AVX优化的版本，要想使用GDB的回退执行可能需要重新编译glibc或者修改动态链接器使它不要使用AVX版本的函数

参考[stackoverflow](https://stackoverflow.com/questions/42451492/disable-avx-optimized-functions-in-glibc-ld-hwcap-mask-etc-ld-so-nohwcap-for)