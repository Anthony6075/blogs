---
title: Linux gdb常用命令
categories:
- computer_science
tags:
- linux
- gdb
- shell
---

# 1. 编译

注意： 

1. **不要使用优化**(即使用-O0选项)

1. **带上调试信息**(即使用-g选项)

-g的调试信息级别是2，使用-g3会带上额外调试信息(如关于宏的)

总而言之，最好使用`-g3 -O0`

# 2. help system

## 2.1 `help`命令

`help`缩写为`h`

`help <command>` 输出某个命令的全部文档

## 2.2 `apropos`命令

`apropos [-v] <regexp>`遍历所有命令、别名和文档查找`<regexp>`的匹配

## 2.3 其他

`complete <args>`输出`<args>`所有可能的命令补全结果

`info`命令用于获取正在调试的程序的状态信息

`show`命令用于获取GDB本身的状态信息

# 3. 启动GDB

## 3.1 正常启动

直接指定二进制文件，例如`gdb myprogram`

或者先不带参数启动GDB再用`file`命令加载，例如

```bash
$ gdb
(gdb) file myprogram
Reading symbols from myprogram...
(gdb) 
```

## 3.2 `-q`或者`--quiet`

启动时不打印GDB版本信息

## 3.3 指定被调试程序的命令行参数

启动后查看程序命令行参数用`show args`，查看当前函数的参数用`info args`

### 3.3.1 `--args`选项

例如`gdb --args myprogram 1 2 3`

### 3.3.2 run命令或者start命令

例如`run 1 2 3`或者`start 1 2 3`

### 3.3.3 `set args`命令

例如`set args 1 2 3`

## 3.4 调试coredump文件

```bash
gdb <executable_file_path> <coredump_file_path>
```

# 4. GDB常用命令

#### 1. `run`和`start`

运行程序，其中`run`的缩写为`r`，`start`会在main函数开始处暂停

#### 2. `break [LOCATION] [thread THREAD_NUM] [if CONDITION]`

缩写为`b`，设置断点，可以使用文件名和行号，也可以使用函数名

- 不指定`[LOCATION]`的话断点打到正要执行的那一行

- `[thread THREAD_NUM]`用于限制只有特定线程hit这个断点时程序才会暂停

- `if`子句用来指定条件断点，只有在`CONDITION`为真时断点才会触发

- `tbreak`命令类似于`break`，只是用来设置临时断点，在第一次hit之后GDB就会自动删除它

#### 3. `continue`

在断点处继续执行，直到下一个断点

#### 4. `next`

缩写为`n`，step over，可跟一个数字参数`[N]`指定执行次数

#### 5. `step`

缩写为`s`，step into，可跟一个数字参数`[N]`指定执行次数

#### 6. `print`

缩写为`p`，打印表达式的值

#### 7. `watch EXP`

监控表达式`EXP`，当它的值改变时暂停程序

#### 8. `backtrace`或者`where`

缩写为`bt`，打印调用栈帧层次

#### 9. `frame`

缩写为`f`，选择并打印栈帧

- 不带参数时，打印当前被选择的栈帧，打印结果为(1)第一行为层次相关，(2)第二行为**此栈帧内**正要执行的代码
- 指定数字参数，选择到该层栈帧并打印
- `frame function <function_name>`或者缩写为`f f <function_name>`，通过函数名选择栈帧并打印

#### 10. `finish`

缩写为`fin`，运行到当前函数返回，step out

#### 11. `delete [BREAKPOINT_NUM]`

- 缩写为`d`，通过断点号`BREAKPOINT_NUM`删除断点

- 也可以跟其他子命令用以删除其他GDB对象

#### 12. `info`

缩写为`i`，用于获取正在调试的程序的状态信息，例如：

- `info breakpoints`或者缩写为`i b`， 查看断点

- `info threads`或者缩写为`i th`，查看线程信息

- `info shared`，查看加载的动态链接库

#### 13. `shell`

将这一行的剩余部分当做shell命令执行，无其余部分时会启动一个shell

#### 14. 清屏

ctrl+L或者`shell clear`

#### 15. `list`

缩写为`l`，列出源代码

- 无参数时，列出上一次调用后挨着的10行

- `list -`，列出上一次调用前挨着的10行

- 指定单参数时，列出该行前后共10行

- 指定逗号分隔的双参数时，它们指定了列出的代码范围，如果一个参数是空，以另一个参数为准向前或向后列出10行

- 行号也可以用函数名指定，如`l f,`列出函数f的前10行，接着调`l`可以不断向后列出函数f的其他代码

- 可以使用`show listsize`或者`set listsize <size>`查看或修改listsize，默认为10行

#### 16. `thread <THREAD_NUM>`

缩写为`t`，切换到线程`<THREAD_NUM>`

#### 17. `until LINE_SPEC`

缩写为`u`，(在当前栈帧)执行下去直到`LINE_SPEC`指定的行，停下来时`LINE_SPEC`是下一行要执行的

- 可以用来将循环执行完毕

- 如果`LINE_SPEC`超出当前函数，只会执行到当前函数返回为止

#### 18. `call`

调用函数并打印返回值，例如`call f(1,2,3)`

#### 19. `return [<return_value>]`

使当前函数(栈帧)直接返回，如果指定了`<return_value>`，将它作为函数返回值

#### 20. `display [EXP]`

- 在每次程序暂停时自动打印表达式`EXP`的值
- 不带参数时，列出当前所有的自动打印表达式

#### 21. `undisplay [NUM]`

- 删除`NUM`标识的自动打印表达式
- 不带参数时，删除所有自动打印表达式

#### 22. `disable [BREAKPOINT_NUM]`

缩写为`dis`，用于disable某个断点

- 不带参数时，disable所有断点
- 也可以使用子命令来disable其他GDB对象，如`disable display [DISPLAY_NUM]`

#### 23. `enable [BREAKPOINT_NUM]`

缩写为`en`，和`disable`命令作用完全相反

#### 24. `clear [LINE_SPEC]`

缩写为`cl`，用于删除某一行上的断点

- 不带参数时，删除当前栈帧的当前行上的断点
- `LINE_SPEC`可以是函数名，删除函数声明行上的断点

#### 25. `set`

- `set VAR = EXP`，计算表达式`EXP`并赋值给变量`VAR`
- 当`VAR`变量名和`set`的子命令名冲突时，可以使用`set var VAR = EXP`子命令来对变量赋值

#### 26. `up [NUM]`

- 选择上层栈帧(调用当前函数的函数)
- `NUM`指定向上走几层，默认为1

#### 27. `down [NUM]`

缩写为`do`

- 选择下层栈帧(当前函数调用的函数)
- `NUM`指定向下走几层，默认为1

#### 28. `set history save on`

在退出时保存命令历史，可以写入`~/.gdbinit`文件以默认启用命令历史保存

#### 29. `pipe`

缩写为`|`，将gdb命令的输出重定向到一个shell命令，使用方式：

`| [COMMAND] | SHELL_COMMAND`或者`pipe [COMMAND] | SHELL_COMMAND`

#### 30. `commands [BREAK_NUM]`

设定在指定断点hit时需要执行的命令

- 无`BREAK_NUM`参数时，默认使用最后设置的断点(即`$bpnum`)

- 如需清空上次`commands`的设置，只需再次调用`commands`并传入空的命令

- 命令第一行输入`silent`可以使该断点hit时不打印输出

#### 31. `ignore <BREAK_NUM> <COUNT>`

忽略断点`BREAK_NUM`接下来的`COUNT`次hit

# 5. 调试多线程程序

## 5.1 All-Stop Mode

这是GDB的默认模式。

当一个线程stop时(比如因为hit断点)，所有线程都stop。

类似地，当继续执行这个线程时(比如使用`step`或者`next`命令)，所有其他线程也会恢复执行，但它们不受当前线程的`step`或者`next`命令控制而自由执行(因为线程调度依赖kernel，而不受GDB控制)，因此可能执行任意条语句。而且当这些其他线程遇到断点时会导致GDB发生自动线程切换，此时原线程的`step`或者`next`命令甚至可能还没执行完。

### 5.1.1 锁住OS scheduler

这是为了在恢复执行时只允许一个线程运行。

scheduler锁定模式有多种，使用命令`set scheduler-locking <mode>`设置，`show scheduler-locking`获取。

可用的模式有：

- `off` 无锁定，任意线程可以任意运行。

- `on` 只有当前线程可以运行，其他线程依然stop。

- `step` 当单步执行时效果类似`on`，使用其他执行命令(如`continue`, `until`, `finish`)时类似`off`。(由于机器不支持此模式，未测试文档中的"单步执行"是不是指`step`和`next`命令)

- `replay` (默认)在replay模式时类似于`on`，在record模式和正常执行时类似`off`。

## 5.2 常用命令

- `p $_thread`，打印当前线程号

- `info threads`或者缩写为`i th`，查看线程信息

- `thread [THREAD_NUM]`或者缩写为`t [THREAD_NUM]`，切换线程

# 6. 方便变量(Convenience Variables)

## 6.1 基本介绍

- 形式为以`$`开头的名字(以`$`开头的数字不是)

- 无固定类型，甚至可以是结构体或者数组

## 6.2 赋值

使用`set`命令，如`set $foo = *object_ptr`

## 6.3 使用示例

不断使用回车遍历数组：

```gdb
set $i = 0
p arr[$i++]
```

# 7. 回退执行

使用命令`record`，`reverse-next`，`reverse-step`，`reverse-continue`，`reverse-finish`等等

但在支持AVX(Advanced Vector Extension)的现代CPU上`record`命令不能使用，因此GDB回退执行的功能很鸡肋

glibc的string相关函数会使用AVX优化的版本，要想使用GDB的回退执行可能需要重新编译glibc或者修改动态链接器使它不要使用AVX版本的函数

参考[stackoverflow](https://stackoverflow.com/questions/42451492/disable-avx-optimized-functions-in-glibc-ld-hwcap-mask-etc-ld-so-nohwcap-for/44468494#44468494)

# 6. TUI(Text User Interface)

## 6.1 GDB tui常用命令

### 1. 启动和退出tui模式

- 分别是`tui enable`(缩写为`tui e`)和`tui disable`(缩写为`tui d`)命令

- 或者使用快捷键ctrl+x+a，按第一次进入，第二次退出

### 2. ctrl+L 刷新屏幕

在程序使用标准输出和标准错误打印时屏幕可能乱掉，可以使用ctrl+L进行刷新

ctrl+L不会被记在命令历史里，下一次ENTER不会重复ctrl+L

### 3. `info win`命令

列出当前显示的所有窗口

### 4. `focus`命令

缩写为`fs`，改变焦点到不同窗口，使用方式`focus [WINDOW-NAME | next | prev]`

### 5. 查看文档

使用`help text-user-interface`查看tui相关所有命令

## 6.2 CGDB

CGDB是GDB的一个控制台前端(Console Frontend)，它还是使用GDB进行实际调试

GDB的tui模式不支持源码窗口和命令窗口垂直分屏，因此使用CGDB

### 6.2.1 使用

#### 1. 启动

在shell中输入`cgdb`启动程序

调用方式类似GDB: 

```bash
cgdb [cgdb options] [--] [gdb options]
```

#### 2. 退出

- 在GDB窗口输入`quit`或者ctrl+D

- 在源代码窗口输入`:quit`

#### 3. 调试

CGDB的操作类似vim，按`ESC`键进入源代码窗口，按`i`进入GDB窗口

#### 4. 垂直分屏

源代码和命令窗口垂直分屏输入：

`:set winsplitorientation=vertical`或者`:set wso=vertical`

#### 5. `~/.cgdb/cgdbrc`文件

类似于`!/.bashrc`文件，用于初始化cgdb，可将`:set wso=vertical`写入以默认垂直分屏

### 6.2.2 资源

- [官网](https://cgdb.github.io/)

- [文档](https://cgdb.github.io/docs/cgdb-split.html)

### 6.2.3 注意

- 目前最新版为0.8.0，没有发正式版

# 7. 其他

## 7.1 获取目标文件的编译flags

为查看某个目标文件编译时是否启用了优化或者携带调试信息(debuginfo)

使用命令`readelf -w <object_file> | grep producer | head -1`查看使用的编译器和编译flags

检查flags里是否有`-O`或者`-g`即可

## 7.2 术语inferior

inferior在GDB里基本等同于进程，它可以包含多个线程

