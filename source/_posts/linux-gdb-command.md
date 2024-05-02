---
title: Linux GDB 常用命令
categories:
- computer_science
- gdb
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

inferior在GDB里基本等同于进程，它可以包含多个线程

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

缩写为`n`，step over，(可跟一个数字参数`[N]`指定重复`next`的次数，但遇到断点还是会提前stop)

#### 5. `step`

缩写为`s`，step into，(可跟一个数字参数`[N]`指定重复`step`的次数，但遇到断点还是会提前stop)

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

- `info display`，查看自动展示表达式

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

# 5. 其他

## 5.1 获取目标文件的编译flags

为查看某个目标文件编译时是否启用了优化或者携带调试信息(debuginfo)

使用命令`readelf -w <object_file> | grep producer | head -1`查看使用的编译器和编译flags

检查flags里是否有`-O`或者`-g`即可

## 5.2 查看复杂程序的编译flags

对于包含多个翻译单元的复杂程序，需要先从readelf的输出中找到指定的翻译单元，再去查看它的producer属性

# 6. 调试器的不足

调试器毕竟是对原程序的一种入侵，可能有不可避免的副作用(例如：如果一个线程因为断点而stop，另一个线程因为syscall而阻塞，那么这个syscall可能还没有真实执行完就会早熟地返回)，再加上GDB本身的使用也很复杂，所以总会有想不到的情况，遇到没见过的不熟悉的问题不要心慌，**要想其他办法绕过去**。再者说，技术细节其实并不重要也没人关心，重要的是**完成实际任务**和**与人打交道**。

