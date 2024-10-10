---
title: memory sanitizer
categories:
- computer_science
- sanitizer
tags:
- linux
- c++
- sanitizer
---

# 1. 简介

MemorySanitizer简称msan，用于发现C/C++程序中对**未初始化内存**的**读取**。

msan是比特精度的。

msan的功能是Valgrind功能的子集，但运行速度更快。

msan使用stderr打印错误报告

msan发现错误即报告，而不是等到程序即将结束时统一报告

msan发现如下bug并报告warning:

1. 在条件分支中使用未初始化的值。

1. 使用未初始化的指针进行内存访问。

1. 函数返回未初始化的值(这是未定义行为，可以使用`-fno-sanitize-memory-param-retval`来disable这项检查)。

1. 传递未初始化数据到一些libc函数中。

# 2. 使用方法

## 2.1 常用使用方法

- 使用clang编译器

- 使用编译选项`-fsanitize=memory -fPIE -pie -fno-omit-frame-pointer -g`

- 默认错误报告是fatal的，即发现并报告第一个错误后就退出，如需发现多个错误需使用编译选项`-fsanitize-recover=memory`

例如如下程序a.cc：

```
#include <iostream>

int main(int argc, char** argv) {
  int* a = new int[10];
  a[5] = 0;

  if (a[argc]) {
	  std::cout << "yes" << std::endl;
  }

  delete[] a;
  return 0;
}
```

编译：

```
$ clang++ -fsanitize=memory -fPIE -pie -fno-omit-frame-pointer -g -fsanitize-recover=memory a.cc
```

运行：

```
$ ./a.out
```

输出：

```
==118555==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x56274f184578 in main /root/msan/a.cc:7:7
    #1 0x7fe1025c6082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #2 0x56274f10932d in _start (/root/msan/a.out+0x1e32d)

SUMMARY: MemorySanitizer: use-of-uninitialized-value /root/msan/a.cc:7:7 in main
MemorySanitizer: 1 warnings reported.
```

## 2.2 简单使用方法

仅使用`-fsanitize=memory`和`-g`两个编译选项。

```
$ clang++ -fsanitize=memory -g a.cc && ./a.out
```

# 3. 追踪内存分配位置

msan可以追踪未初始化值的内存分配位置并报告。

启用该追踪需使用编译选项`-fsanitize-memory-track-origins`。

该追踪会额外减慢程序速度1.5x~2.5x。

例如对于上面的程序a.cc：

```
$ clang++ -fsanitize=memory -fPIE -pie -fno-omit-frame-pointer -g -fsanitize-recover=memory -fsanitize-memory-track-origins a.cc
$ ./a.out
==118562==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x5596e037f85b in main /root/msan/a.cc:7:7
    #1 0x7f6a6230d082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #2 0x5596e030432d in _start (/root/msan/a.out+0x1e32d)

  Uninitialized value was created by a heap allocation
    #0 0x5596e037d109 in operator new[](unsigned long) (/root/msan/a.out+0x97109)
    #1 0x5596e037f666 in main /root/msan/a.cc:4:12
    #2 0x7f6a6230d082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: MemorySanitizer: use-of-uninitialized-value /root/msan/a.cc:7:7 in main
MemorySanitizer: 1 warnings reported.
```

# 4. 注意

为了更好地使用msan，程序中的**所有代码**都需要被msan插桩(使用`-fsanitize=memory`编译)，包括链接的库(尤其是c++标准库libc++)。

否则未插桩的代码会导致msan报告假阳性。

参考[该文档](https://github.com/google/sanitizers/wiki/MemorySanitizerLibcxxHowTo)使用msan构建c++标准库。

# 5. 参考文献

- https://github.com/google/sanitizers/wiki/MemorySanitizer
