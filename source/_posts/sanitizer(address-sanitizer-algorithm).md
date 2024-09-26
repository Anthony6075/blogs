---
title: address sanitizer算法原理
categories:
- computer_science
- sanitizer
tags:
- linux
- c++
- sanitizer
---

# 1. 算法原理精简版(TL;DR)

## 1.1 运行时库

asan运行时库提供了asan版的malloc和free函数。

1. malloc函数在分配的内存区域周围会再分配redzones内存并在redzones内存投毒(poisoned)。

2. free函数对被释放的内存区域投毒(poisoned)，并将它放到临时隔离队列(quarantine queue)中，使它在一段时间内不会被再malloc。

## 1.2 插桩

程序中的内存访问操作会做如下转换：

转换前：

```
*address = ...;  // or: ... = *address;
```

转换后：

```
if (IsPoisoned(address)) {
  ReportError(address, kAccessSize, kIsWrite);
}
*address = ...;  // or: ... = *address;
```

# 2. 内存映射

## 2.1 内存划分

整个内存虚拟地址空间被分为不相交的两类：

1. Mem：被常规应用代码使用的内存区域。

2. Shadow：用于保存元数据(称为shadow values)。

## 2.2 内存映射

Mem区域和Shadow区域存在映射关系，Mem对应的具体shadow value表示了其有没有被投毒(poisoned)。

8字节(1个qword)的Mem对应1字节的Shadow。之所以以8字节为单位映射Mem，是因为malloc返回的内存块都是对齐到8字节的。

这个1字节shadow value的可能取值及含义包括：

1. 0。这8字节Mem都可用(即未投毒，unpoisoned，addressable)。

2. 负值。这8字节Mem都不可用(即都被投毒，poisoned，not addressable)。

3. [1, 7]之间的正整数`k`。前`k`字节可用未投毒，后`8-k`字节被投毒不可用。这种情况只会发生在malloc分配的内存区域尾部。

## 2.3 具体映射关系

映射函数`MemToShadow`具体为：

- 64位平台上：`Shadow = (Mem >> 3) + 0x7fff8000;`。

- 32位平台上：`Shadow = (Mem >> 3) + 0x20000000;`。

# 3. 插桩

插桩代码可进一步表示为：

```
byte *shadow_address = MemToShadow(address);
byte shadow_value = *shadow_address;
if (shadow_value) {
  if (SlowPathCheck(shadow_value, address, kAccessSize)) {
    ReportError(address, kAccessSize, kIsWrite);
  }
}

// Check the cases where we access first k bytes of the qword
// and these k bytes are unpoisoned.
bool SlowPathCheck(shadow_value, address, kAccessSize) {
  last_accessed_byte = (address & 7) + kAccessSize - 1;
  return (last_accessed_byte >= shadow_value);
}
```

需要注意`MemToShadow(ShadowAddr)`会落到`ShadowGap`区域，`ShadowGap`是unaddressable，因此在程序代码里直接访问`ShadowAddr`会导致crash。

# 4. 运行时库

asan运行时库提供了自己的`malloc`/`free`函数和错误报告函数(如`__asan_report_load8`)。

1. `malloc`函数除了分配指定大小的内存区域，还在这块区域周围分配了redzones内存；前者没有投毒，后者会被投毒。

2. `free`函数对整块内存投毒，并且将这块内存放入临时隔离队列(quarantine queue)，这样一段时间内这块内存不会再被malloc返回。

# 5. stack相关的bug发现

## 5.1 stack-buffer-overflow

定义栈上buffer时在它的周围插入redzones并poison，函数返回时unpoison。

## 5.2 stack-use-after-return

发现stack-use-after-return错误需分配fake stack以将stack转换为heap，因此它的算法原理类似于发现heap-use-after-free。

## 5.3 stack-use-after-scope

在定义本地变量时unpoison它的内存，在定义所在的scope的尾部poison它的内存。

# 6. 未对齐的内存访问

当前的内存映射实现不会发现未对齐的部分越界访问，例如：

```
int *x = new int[2]; // 8 bytes: [0,7].
int *u = (int*)((char*)x + 6);
*u = 1;  // Access to range [6-9]
```

# 7. 参考文献

- https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm

- https://github.com/google/sanitizers/wiki/AddressSanitizerUseAfterReturn

- https://github.com/google/sanitizers/wiki/AddressSanitizerUseAfterScope