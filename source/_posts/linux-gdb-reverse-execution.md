---
title: Linux GDB反向执行
categories:
- computer_science
- gdb
tags:
- linux
- gdb
- shell
---

# 1. 应用场景

反向执行(Reverse Execution)应用在：当发现执行太过了的时候，关注的问题可能已经发生了，需要反向执行。

需要目标环境支持，不是所有目标环境都可以实现反向执行的。

# 2. 程序执行的记录(record)和重放(replay)

record会拖慢程序执行速度。

如果执行日志中包括下一条指令的执行记录，GDB会使用replay mode，此时运行程序不会真正执行代码，而是从执行日志中恢复程序状态。

否则如果执行日志中不包括下一条指令的执行记录，GDB会使用record mode，此时运行程序会正常执行，但GDB会记录执行日志以备将来replay。

即使程序运行的平台不支持Reverse Execution，在执行日志记录的范围内，GDB也可以通过replay支持Reverse Execution。

在反向执行时，如果执行日志包括先前指令的记录，GDB会使用replay mode，否则，如果平台支持反向执行，GDB会使用record mode，如果不支持则stop。

# 3. record和replay相关的命令

## 3.1 `record [METHOD]`

缩写为`rec [METHOD]`。

启动process record and replay target。`record`命令需要在调用`run`或者`start`之后调用。`METHOD`参数可能的取值为：

1. `full`。默认值；由GDB软件实现；支持反向执行；不支持Non-Stop Mode。

1. `btrace <format>`。依赖硬件支持；允许有限的反向执行。

## 3.2 `record stop`

缩写为`rec s`。

停止process record and replay target，执行日志将被删除，进程或者终止或者保持在最后状态。

进程退出时该target会自动stop，重新调试需要再调`record`命令启动target。

## 3.3 `info record`

缩写为`i rec`，展示记录的各种信息。

对于`METHOD`为`full`时，展示信息包括：

1. 当前处于record mode还是replay mode；

1. 记录的最小和最大的指令号；

1. 执行日志包含的指令数量；

1. 等等。

## 3.4 `record delete`

缩写为`rec d`。

在replay mode起作用，删除后续的执行日志并从当前开始记录新的执行日志，即抛弃过去记录的旧未来，开始创建并记录一个新未来。

# 4. 反向执行相关的命令

## 4.1 `reverse-continue [ignore-count]`

缩写为`rc [ignore-count]`，反向continue。

类似continue，遇到断点会停下来。

## 4.2 `reverse-step [count]`

缩写为`rs [count]`，反向step。

类似step，上一行是函数调用的话会进入函数并stop在最后一条语句处。

## 4.3 `reverse-next [count]`

缩写为`rn [count]`，反向next。

类似next，如果当前已经在某函数第一行会导致回到它的caller。

## 4.4 `reverse-finish`

无缩写，反向执行直到当前函数的开始处，就像finish会执行到当前函数的结束处。

## 4.5 `set exec-direction [ reverse | forward ]`

设置命令执行方向，受影响的命令包括`step`，`stepi`，`next`，`nexti`，`continue`，`finish`。注意`return`命令不能反向工作。

1. `forward`是默认值，正常向前执行。

2. `reverse`时上述受影响命令会自动反向执行。

# 5. 启用record和反向执行

在启用AVX(Advanced Vector Extension)的现代CPU上`record`命令不能使用，glibc的string相关函数(`strlen`，`memset`等等)会使用AVX优化的版本，一般而言，要想使用GDB的反向执行，可能需要重新编译glibc或者修改动态链接器使它不要使用AVX版本的函数。

但是最近版本的glibc增加了tunables的功能可以容易地避开AVX，使用环境变量`GLIBC_TUNABLES=glibc.cpu.hwcaps=-AVX2`来调用GDB，例如：

```bash
GLIBC_TUNABLES=glibc.cpu.hwcaps=-AVX2 gdb ./my_program
```

进入GDB后先调用`start`，再调用`record`，然后开始调试即可。

也可以在shell中配置一下gdb命令别名自动完成上述工作，如下：

```bash
alias rgdb='GLIBC_TUNABLES=glibc.cpu.hwcaps=-AVX2 gdb -q -ex start -ex record'
```

接着只需简单调用`rgdb`即可：

```bash
rgdb ./my_program
```

# 6. GDB的一个小bug

启用record后，在被调试进程运行完毕时，GDB提示是否真地退出程序：

```
The next instruction is syscall exit_group.  It will make the program exit.  Do you want to stop the program?([y] or n)
```

但是输入`n`(即no)才会退出。

# 7. 参考

1. GDB文档

    1. [Running programs backward](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Reverse-Execution.html#Reverse-Execution)

    1. [Recording Inferior’s Execution and Replaying It](https://sourceware.org/gdb/current/onlinedocs/gdb.html/Process-Record-and-Replay.html#Process-Record-and-Replay)

1. [stackoverflow](https://stackoverflow.com/a/61048314/18781047)

1. [GDB tunables](https://gotplt.org/posts/the-story-of-tunables.html)