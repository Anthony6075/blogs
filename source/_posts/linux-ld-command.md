---
title: Linux ld命令使用方法
date: 2024-05-05 15:36:01
categories:
- computer_science
- linker
tags:
- linux
- shell
- linker
---

# 1. ld命令: linux链接器

作用: 将一系列目标文件(.o)和静态库文件(.a)链接起来(动态库文件.so也可能作为ld的输入文件)

# 2. 常用选项

注意：对于单字母的选项(如`-l`等)，选项的参数直接跟在选项之后，用不用空格分隔都可以。

## 2.1 `-l <namespec>`或者`--library=<namespec>`

使用`<namespec>`标识的库文件(动态库lib<namespec>.so优先于静态库lib<namespec>.a)。

如果`<namespec>`是`:filename`的形式，查找文件名为`filename`的库文件。

## 2.2 `-L <searchdir>`或者`--library-path=<searchdir>`

添加一个查找库文件(和控制脚本)的目录`<searchdir>`以供`-l`选项使用，多个此选项之间的顺序关系为在命令行出现的顺序，最后为默认查找目录。

默认查找目录是系统依赖的，较为复杂，如/lib/x86_64-linux-gnu。

## 2.3 `-rpath=<dir>`

添加`<dir>`到此次生成的动态库或可执行文件的运行时动态库查找路径，以供动态链接器使用。

## 2.4 `-shared`

表明此次链接要创建一个共享库。

## 2.5 `-static`

与`-shared`不同，`-static`表示其之后的`-l`选项**只选择静态库**去参与链接。

## 2.6 `-( <archives> -)` `--start-group <archives> --end-group`

`<archives>`指定的库文件会被重复查找以解析未定义符号，而非按命令行出现顺序只查找一次，此选项可能会带来较大性能开销。

## 2.7 `--whole-archive`和`--no-whole-archive`

`--whole-archive`之后的.a静态库中的所有目标文件都会被链接进来，而不是像普通情况下只链接其中用到的目标文件。

这个选项常被用来将静态库转为动态库。

使用`--whole-archive`后不要忘记使用`--no-whole-archive`取消后续作用。

## 2.8 `--version-script=<version-scriptfile>`

指定version script，常常在创建共享库时使用。可以用来做共享库的符号隐藏，脚本文件内容示例如下:


```
{
  global:
    extern "C++" {
        my_namespace::*;
        my_prefix_*;
    };
  local: *;
};
```

上述文件中标记为global的符号会被导出，其他标记为local的符号不可见。

关于共享库的符号导出，GNU有[文章](https://www.gnu.org/software/gnulib/manual/html_node/Exported-Symbols-of-Shared-Libraries.html "Controlling the Exported Symbols of Shared Libraries")介绍。

# 3. 注意事项

1. 被链接为动态库的目标文件需要在编译时指定`-fPIC`选项，在x86平台上`-fPIC`(大写字母)和`-fpic`(小写字母)没有区别

2. 静态库.a只是一些目标文件.o的集合外加一些索引信息; 从ld的视角，静态库像一个“半成品”，动态库是“最终产品(final product)”，因为不能merge两个动态库成一个动态库，但静态库可以任意merge为新的静态库或者动态库

3. 当使用编译器驱动器去调用ld时，传递给ld的命令行选项应该使用前缀`-Wl,`

4. 使用编译器驱动器(compiler driver, 如gcc或者g++)去调用ld，尽量不要直接调用，否则可能有未指定到的选项