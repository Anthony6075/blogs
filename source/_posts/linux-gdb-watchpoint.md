---
title: Linux GDB Watchpoint
categories:
- computer_science
tags:
- linux
- gdb
- shell
---

# 1. 基本概念

watchpoint又称data breakpoint，用于监控某个表达式，当它的值改变时stop程序。

表达式可以是单个变量，也可以包含操作符(任意原语言中有效的操作符)。

# 2. 分类

# 2.1 software watchpoint

GDB单步执行程序，然后每次都计算表达式值。因此程序执行速度可能慢了数百倍，而且它在被监控值改变后的下一条语句处才报告值改变，而非在确切的CPU指令处。

# 2.2 hardware watchpoint

依赖硬件支持(如x86架构)，不会减慢程序执行速度，且在改变被监控值的CPU指令执行时即能及时报告。

GDB会尽可能创建hardware watchpoint。

有时GDB不能创建hardware watchpoint，可能的原因包括：

1. 要监控的内存区域过大。(此时GDB可能把它拆分成多个小的hardware watchpoint)。

1. 已经设置了太多hardware watchpoint。但可能到程序resume执行时(而不是创建watchpoint时)GDB才会报告"Hardware watchpoint <num>: Could not insert watchpoint"，此时需要delete或者disable一些hardware watchpoint。

# 3. 创建watchpoint

```gdb
watch [-l|-location] expr [thread thread-id]
```

为表达式`expr`创建监控，当它的值改变时stop程序。

`-location`表明：`expr`计算的结果需要是一个左值，GDB会取它的地址并监控这个地址上的内容变化，它的类型决定了监控的内存大小。(如果`expr`本身是一个左值，有无`-location`都一样；如果`expr`是右值，带`-location`会报错，因为它不能取地址)

`thread thread-id`表明：只有特定的线程`thread-id`改变`expr`的值，程序才会stop。这种限制线程的watchpoint只适用于hardware watchpoint。

# 4. 查看watchpoint

- `info breakpoints`或者缩写为`i b`，展示所有断点，包括watchpoint

- `info watchpoints`，展示所有watchpoint

# 5. 删除watchpoint

## 5.1 主动删除

- `delete [BREAK_NUM]`或者缩写为`d [BREAK_NUM]`

## 5.2 自动删除

当本地变量`local_var`离开作用域时，含有`local_var`的`expr`的watchpoint都会被自动删除。

# 6. 多线程程序中的watchpoint

按理说，GDB应该发现来自每个线程的对`expr`(被监控表达式)的改变。

对hardware watchpoint来说，GDB确实可以发现来自所有线程的修改。

对software watchpoint来说，GDB只能监控单线程中的`expr`。如果你自信`expr`只能被当前线程修改且不会发生线程切换，然后可以照常使用software watchpoint。然而，GDB可能不会注意到对`expr`的来自非当前线程的修改。

即：在多线程程序中，software watchpoint的作用有限。

# 7. 监控复杂类对象

应该还是要先知道它的数据成员，拆成基本数据类型来监控。例如监控`std::string`类型的字符串变量`s`的内容变化，需要先拿到`s`的`char*`指针成员再监控该指针指向的内存，直接`watch s`(也可以但)不准确。