---
title: Linux GDB Convenience Variables
date: 2024-05-01 19:43:35
categories:
- computer_science
- gdb
tags:
- linux
- gdb
- shell
---

# 1. 基本介绍

- 形式为以`$`开头的名字(以`$`开头的数字不是)

- 无固定类型，甚至可以是结构体或者数组

# 2. 赋值

使用`set`命令，如`set $foo = *object_ptr`

# 3. 使用示例

不断使用回车遍历数组：

```gdb
set $i = 0
p arr[$i++]
```