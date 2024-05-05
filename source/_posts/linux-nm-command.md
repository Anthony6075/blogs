---
title: Linux nm命令常见用法
categories:
- computer_science
- linker
tags:
- linux
- shell
- linker
---

# 1. nm命令

单词name的简称，用于查看二进制目标文件中的符号信息

# 2. 输出

三列，分别是(1)符号地址(2)符号类型(3)符号名

常见符号类型包括(1)"T"，"t"，代码段中的符号(2)"U"(大写字母u)，未定义的符号

# 3. 常用选项

## 3.1 `-A`

在每一行都显示文件名

## 3.2 `-C`

符号名demangle

## 3.3 `-u`，小写字母u

只显示未定义的符号