---
title: sanitizer工具
date: 2024-10-07 18:05:41
categories:
- computer_science
- sanitizer
tags:
- linux
- c++
- sanitizer
---

# 1. Sanitizer系列简介

中文直译：
sanity n. 神志正常，精神健全
sanitize v. 消毒
sanitizer n. 消毒剂

# 2. 包含的工具

## 2.1 AddressSanitizer

用于发现C/C++程序中的内存错误，比如use-after-free和{heap,stack,global}-buffer overflow。

## 2.2 LeakSanitizer

用于发现内存泄漏，已被集成到AddressSanitizer中。

## 2.3 ThreadSanitizer

用于发现C/C++和Go程序中的数据竞争(data race)。

## 2.4 MemorySanitizer

用于发现未初始化内存的使用。

# 3. 注意

- 并不是一定能检测出所有的bug ! ! !

- asan/tsan/msan在同一个程序中最多使用一个

# 4. 参考文献

- https://github.com/google/sanitizers/wiki

- https://clang.llvm.org/docs/UsersManual.html#cmdoption-f-no-sanitize
