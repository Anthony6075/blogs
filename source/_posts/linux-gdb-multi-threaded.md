---
title: Linux GDB 多线程调试
date: 2024-05-02 07:12:55
categories:
- computer_science
- gdb
tags:
- linux
- gdb
- shell
---

# 1. All-Stop Mode

这是GDB的默认模式(共两个模式，另一个为Non-Stop Mode)。

当一个线程stop时(比如因为hit断点)，所有线程都stop。

类似地，当继续执行这个线程时(比如使用`step`或者`next`命令)，所有其他线程也会恢复执行，但它们不受当前线程的`step`或者`next`命令控制而自由执行(因为线程调度依赖kernel，而不受GDB控制)，因此可能执行任意条语句。而且当这些其他线程遇到断点时会导致GDB发生自动线程切换，此时原线程的`step`或者`next`命令甚至可能还没执行完。

## 1.1 锁住OS scheduler

这是为了在恢复执行时只允许一个线程运行。

scheduler锁定模式有多种，使用命令`set scheduler-locking <mode>`设置，`show scheduler-locking`获取。

可用的模式有：

- `off` 无锁定，任意线程可以任意运行。

- `on` 只有当前线程可以运行，其他线程依然stop。

- `step` 当单步执行时效果类似`on`，使用其他执行命令(如`continue`, `until`, `finish`)时类似`off`。(由于机器不支持此模式，未测试文档中的"单步执行"是不是指`step`和`next`命令)

- `replay` (默认模式)在replay模式时类似于`on`，在record模式和正常执行时类似`off`。

# 2. libthread_db

GDB使用libthread_db库(字母db指debug)来进行线程调试，它会自动查找和libpthread库相匹配的libthread_db库并初始化，如果不能找到libthread_db，线程调试会被disable。

在使用`start`或者`run`命令启动程序时，GDB会打印libthread_db的信息。

即便程序只有一个主线程，GDB依然使用libthread_db来enable线程调试。

# 3. 相关命令

- `p $_thread`，打印当前线程号。

- `info threads [thread-id-list]`或者缩写为`i th [ids]`，查看所有线程或指定的线程。

- `thread [THREAD_NUM]`或者缩写为`t [THREAD_NUM]`，切换线程。

- `thread name [NAME]`，给当前线程设置名字，空的`NAME`会删除已存在的名字。

- `thread apply [thread-id-list | all] <command>`，在一些线程上执行命令。(不常用)