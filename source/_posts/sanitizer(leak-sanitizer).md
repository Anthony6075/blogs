---
title: leak sanitizer
date: 2024-09-27 14:55:16
categories:
- computer_science
- sanitizer
tags:
- linux
- c++
- sanitizer
---

# 1. 简介

- 简称LSan

- 用于发现内存泄漏

- 已集成到asan中

- 在性能关键的场景中，LSan也可以在无asan插桩时使用

# 2. 使用方法

## 2.1 using lsan on top of asan

像使用asan那样使用，例如a.c存在内存泄漏问题，代码如下：

```
#include <stdlib.h>

void *p;

int main() {
  p = malloc(7);
  p = 0; // The memory is leaked here.
  return 0;
}
```

使用如下方式编译运行：

```
$ clang -fsanitize=address -g a.c -o a && ./a
```

如果使用asan编译的程序不需检查内存泄漏(只需检查其他asan bugs)，则使用运行时flag`ASAN_OPTIONS=detect_leaks=0`。

## 2.2 lsan单独模式

如果只需检查内存泄漏且不想被asan拖慢运行速度，可以使用编译时flag`-fsanitize=leak`而不是`-fsanitize=address`，此时不会发生编译时插桩。

例如：

```
$ clang -fsanitize=leak -g a.c -o a && ./a
```

需要注意的是，相对于lsan单独模式，官方对于using lsan on top of asan的测试更加充分。

# 3. 运行时flags

所有flags查看[lsan官方文档](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer#flags)。

运行时flags使用环境变量`LSAN_OPTIONS`指定。

常用运行时flags包括：

| flag | 默认值 | 描述 |
| ---- | ---- | ---- |
| exitcode | 23 | 如果非0，程序若存在内存泄漏则退出码为该值 |
| max_leaks | 0 | 如果非0，lsan只报告指定数量的top内存泄漏 |
| suppressions | (none) | 保存忽略规则的文件路径 |

# 4. 忽略某些内存泄漏

使用运行时flag`suppressions=s.txt`指定保存忽略规则的文件s.txt，该文件中一行对应一条规则(也可以用行首的`#`号表示注释)。

每条忽略规则形式为`leak:<pattern>`。

`<pattern>`将对某条内存泄漏的stack trace进行子字符串匹配，如果函数名、源文件名或者二进制文件名匹配，这条内存泄漏会被忽略。

`<pattern>`中也可以使用特殊符号`^`和`$`来匹配字符串开始及结束。

# 5. 参考文档

- https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer
