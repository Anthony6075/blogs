---
title: Linux生成动态库
categories:
- computer_science
- linker
tags:
- linux
- shell
- linker
---

# 1. `-fPIC`选项

所有作为输入的目标文件和静态库在编译时必须添加了`-fPIC`选项，才能生成动态库。

# 2. 生成动态链接库

使用`g++`命令把多个静态库和目标文件生成动态库

```
g++ -Wl,--whole-archive -l:<libfirst.a> -l:<libsecond.a> ...
    -Wl,--no-whole-archive
    -l:<libthird.a> ...
    <object_first.o> <object_second.o> ...
    -Wl,-L<lib_path>
    -shared
    -o <libshared.so>
```

1. `-Wl,--whole-archive`后指定需要包括其中所有目标文件的库

1. `-Wl,--no-whole-archive`用以取消`-Wl,--whole-archive`的后续作用

1. `-Wl,-L<lib_path>`用以添加库文件查找目录

1. `-shared`表明此次链接要创建一个动态库

1. `-o <libshared.so>`指定要创建的动态库文件名