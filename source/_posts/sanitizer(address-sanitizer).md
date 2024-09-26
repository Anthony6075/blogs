---
title: address sanitizer
categories:
- computer_science
- sanitizer
tags:
- linux
- c++
- sanitizer
---

# 1. 简介

AddressSantizer简称为asan。

用于发现C/C++程序中的内存错误。

asan包括两部分：

1. 静态插桩模块(compiler instrumentation module, 目前实现是一个LLVM pass)

1. 运行时库(提供了asan版本的malloc、free等函数)

插桩(instrumentation)即修改程序以便于对它进行分析。

编译器插桩技术是一种在源代码被编译为可执行代码的过程中，向代码中添加额外的指令或元数据的方法。

# 2. 使用方法

## 2.1 编译器版本要求

llvm 3.1及以上，gcc 4.8及以上

## 2.2 常见使用过程

编译：

```
$ clang -fsanitize=address -O1 -fno-omit-frame-pointer -g use-after-free.c -o use-after-free
```

运行：

```
$ ./use-after-free
```

即：

- 使用clang编译器。

- 编译时指定`-fsanitize=address`选项。

- 为了更高的性能，编译时指定`-O1`或者更高的优化等级。

- 为了错误信息中更好的stack traces，编译时指定`-fno-omit-frame-pointer`选项。

- 为了使stack traces里显示函数名和文件名，而不只是函数地址，需要：(1)编译时，指定`-g`选项；（2）运行时，`llvm-symbolizer`命令需在`PATH`环境变量中，或者使用`ASAN_SYMBOLIZER_PATH`环境变量指定`llvm-symbolizer`命令的路径。

## 2.3 简单使用过程

仅使用`-fsanitize=address`和`-g`两个编译选项。

```
$ clang -fsanitize=address -g a.c && ./a.out
```

# 3. 编译时flags和运行时flags

所有flags查看[asan flags官方文档](https://github.com/google/sanitizers/wiki/AddressSanitizerFlags)

## 3.1 编译时flags

常用编译时flags：

| flag | 描述 |
| ---- | ---- |
| `-fsanitize=address` | 启用asan |
| `-fno-omit-frame-pointer` | Leave frame pointers. Allows the fast unwinder to function properly. |
| `-fsanitize-blacklist=path` | 用于关闭部分编译时插桩 |

## 3.2 运行时flags

大部分运行时flags通过环境变量`ASAN_OPTIONS`传递，例如：

```
ASAN_OPTIONS=verbosity=1:malloc_context_size=20 ./a.out
```

为了确定特定版本的asan支持的运行时flag，可以按如下方式运行binary：

```
ASAN_OPTIONS=help=1 ./a.out
```

# 4. 调用栈(stack traces)

asan在下面事件发生时打印调用栈：

- `malloc`和`free`函数调用时

- 线程创建时

- failure

默认情况下，asan使用`llvm-symbolizer`命令来符号化调用栈中的地址(将其转为函数名和在源文件中的位置)。因此运行时`llvm-symbolizer`命令需在`PATH`环境变量中，或者使用`ASAN_SYMBOLIZER_PATH`环境变量指定`llvm-symbolizer`命令的路径。

# 5. 关闭部分插桩

可以使asan忽略某些函数，以加速程序运行，比如这些函数：

- 非常热且已知正确的函数

- 底层且并不关心的函数

- 具有已知问题的函数

在Clang (3.3+)和GCC (4.8+)可以使用属性`no_sanitize_address`使asan忽略某个函数，如：

```
#if defined(__clang__) || defined (__GNUC__)
# define ATTRIBUTE_NO_SANITIZE_ADDRESS __attribute__((no_sanitize_address))
#else
# define ATTRIBUTE_NO_SANITIZE_ADDRESS
#endif
...
ATTRIBUTE_NO_SANITIZE_ADDRESS
void ThisFunctionWillNotBeInstrumented() {...}
```

也可以使用编译flag`-fsanitize-blacklist=my_ignores.txt`通过黑名单文件指定要忽略的函数，如：

```
# 明确忽略该函数
fun:MyFooBar
# 忽略包含MyFooBar的函数
fun:*MyFooBar*
```

# 6. gdb调试asan程序

在gdb中使用**常规方式**调试使用asan编译的程序即可。

当asan发现了bug，它会调用`__asan_report_{load,store}{1,2,4,8,16}`中的某个函数，这些函数内部会调用`__asan::ReportGenericError`。

如果想要gdb在asan报告错误**之前**stop，只需在`__asan::ReportGenericError`设置断点。

如果想要gdb在asan报告错误**之后**stop，只需在`__sanitizer::Die`设置断点，或者使用`ASAN_OPTIONS=abort_on_error=1`。

在gdb中还可以使用如下方式让asan描述某个内存位置：

```
(gdb) set overload-resolution off
(gdb) p __asan_describe_address(0x7ffff73c3f80)
0x7ffff73c3f80 is located 0 bytes inside of 10-byte region [0x7ffff73c3f80,0x7ffff73c3f8a)
freed by thread T0 here: 
...
```

# 7. 可以发现的内存bug

可以发现的内存bug包括：

1. heap-use-after-free

2. heap-buffer-overflow

3. stack-buffer-overflow

4. global-buffer-overflow

5. stack-use-after-return

6. stack-use-after-scope

7. initialization-order-fiasco

8. memory leaks

## 7.1 heap-use-after-free

Use after free也称dangling pointer dereference，悬垂指针。

即使用已经被`delete`的指针。

例如：

```
int main(int argc, char **argv) {
  int *array = new int[100];
  delete [] array;
  return array[argc];  // BOOM
}
```

## 7.2 heap-buffer-overflow

Heap buffer overflow即访问到分配的内存区域的外面。

例如：

```
int main(int argc, char **argv) {
  int *array = new int[100];
  array[0] = 0;
  int res = array[argc + 100];  // BOOM
  delete [] array;
  return res;
}
```

## 7.3 stack-buffer-overflow

Stack buffer overflow即访问到栈上buffer的外面。

例如：

```
int main(int argc, char **argv) {
  int stack_array[100];
  stack_array[1] = 0;
  return stack_array[argc + 100];  // BOOM
}
```

## 7.4 global-buffer-overflow

Global buffer overflow即访问到全局buffer的外面。

例如：

```
int global_array[100] = {-1};
int main(int argc, char **argv) {
  return global_array[argc + 100];  // BOOM
}
```

## 7.5 stack-use-after-return

Use after return即使用已经返回的函数的本地变量。

例如：

```
int *ptr;
__attribute__((noinline))
void FunctionThatEscapesLocalObject() {
  int local[100];
  ptr = &local[0];
}

int main(int argc, char **argv) {
  FunctionThatEscapesLocalObject();
  return ptr[argc];
}
```

默认情况下asan不发现这类stack-use-after-return错误，但它仍可能偶尔发现此类错误却报告为stack-buffer-overflow。

使用`ASAN_OPTIONS=detect_stack_use_after_return=1`使asan发现此类错误。

发现stack-use-after-return在CPU和RAM消耗上比较昂贵，官方benchmark表明最多可造成2x slowdown。

注意：发现stack-use-after-return需要在heap上分配fake stack，因此可能对low-level code带来兼容性问题，比如对函数的本地变量取地址可能发现它不在栈上。

## 7.6 stack-use-after-scope

Use after scope即在定义某个局部变量的作用域之外使用它。

例如：

```
volatile int *p = 0;

int main() {
  {
    int x = 0;
    p = &x;
  }
  *p = 5;
  return 0;
}
```

这个检查默认是启用的，如果不需要可以在编译时指定`-fno-sanitize-address-use-after-scope`。

## 7.7 initialization-order-fiasco

位于不同翻译单元的全局变量的初始化顺序是不确定的。

如果位于x.cpp中的全局变量x的初始化依赖于位于y.cpp中的全局变量y的初始化，但x却先于y执行初始化，就会出现bug。

这种bug不一定必现，因此难以定位。

例如：

a.cc代码如下：

```
#include <cstdio>

extern int y;

int __attribute__((noinline)) read_y() {
  return y;
}

int x = read_y() + 1;

int main() {
  printf("%d\n", x);
  return 0;
}
```

b.cc代码如下：

```
int foo() {
  return 42;
}

int y = foo();
```

不同的编译命令得到的结果不同：

```
$ clang++ a.cc b.cc && ./a.out
1
$ clang++ b.cc a.cc && ./a.out
43
```

这种问题称为[static initialization order problem](https://isocpp.org/wiki/faq/ctors#static-init-order)。

解决方案为[Construct On First Use Idiom](https://isocpp.org/wiki/faq/ctors#static-init-order-on-first-use)，即：将全局变量包装到一个函数内部，成为该函数的static local变量，这样它会且只会在第一次使用时被构造，但需要注意[该static local变量需要使用指针](https://isocpp.org/wiki/faq/ctors#construct-on-first-use-v2)。

### 7.7.1 启用检查

asan默认不启用此类bug的检查，若想启用需指定如下运行时flag。

1. 宽松检查

启用宽松检查需使用运行时flag`ASAN_OPTIONS=check_initialization_order=true`。

如果某个翻译单元中的某个全局变量的初始化，访问了另一个翻译单元中的**未初始化的需动态初始化**的全局变量，宽松检查会进行报告。

2. 严格检查

启用严格检查需指定两个运行时flags`ASAN_OPTIONS=check_initialization_order=true:strict_init_order=true`。

如果某个翻译单元中的某个全局变量的初始化，访问了另一个翻译单元中的**需动态初始化的**全局变量，严格检查会进行报告。

也就是说，宽松检查只会报告真正存在的问题，但严格检查也会报告潜在的问题。

### 7.7.2 假阳性

1. 宽松检查

对于初始化之前仍然能安全访问(例如，它的构造器do nothing的情况)的需动态初始化的全局变量，宽松检查的报告可能是假阳性的。

2. 严格检查

如果访问的另一个翻译单元中的动态初始化的全局变量已经被初始化了(而且编程者对此已有预期)，或者编程者已经通过某种方式(例如使用动态库)保证了想要的特定的初始化顺序，但严格检查仍然会报告错误，此时就是假阳性。

### 7.7.3 忽略某些全局变量的检查

使用编译选项`-fsanitize-blacklist=path/to/blacklist.txt`忽略某些全局变量的初始化顺序bug检查。

`blacklist.txt`文件格式如下：

```
# 单个变量
global:bad_variable=init
# 某个类型的全部变量
type:Namespace::ClassName=init
# 给定文件中的全部变量
src:path/to/bad/files/*=init
```

### 7.7.4 性能影响

初始化顺序bug检查会拖慢程序启动速度。

它的复杂度为O(NM)，其中N是二进制可执行文件中需动态初始化的全局变量的数量，M是翻译单元的数量。

### 7.7.5 注意

根据[asan官方文档](https://github.com/google/sanitizers/wiki/AddressSanitizerInitializationOrderFiasco)学习并跑InitializationOrderFiasco样例过程中发现的奇怪现象：

1. 指定asan相关编译选项后，不同翻译单元的初始化顺序可能改变

1. 宽松检查和严格检查似乎没有区别(即使没有指定`strict_init_order=true`似乎也是严格检查，可能是版本问题)

1. 如果x的初始化依赖于y的初始化，那么`-fsanitize-blacklist`中指定x或者y，错误报告都会disable

1. 文档中忽略变量使用`-fsanitize-ignorelist`，但实验证明`-fsanitize-blacklist`也可，且较低版本clang**支持后者**不支持前者

## 7.8 memory leaks

内存泄漏。

见Leak Sanitizer。

## 7.9 总结

若想启用尽量多的检查，需要额外指定的flags包括：

1. 编译时flags

2. 运行时flags

- `ASAN_OPTIONS=detect_stack_use_after_return=1`启用`stack-use-after-return`

- `ASAN_OPTIONS=check_initialization_order=true:strict_init_order=true`启用`initialization-order-fiasco`

# 8. 注意

## 8.1 error report输出

- asan使用stderr输出error report。

- asan发现错误立即报告，内存泄漏lsan只在进程最后才报告。

- 每一条error report以`==========`行开头。

## 8.2 虚拟内存消耗

asan会消耗很大的虚拟内存(x86_64 linux上约为20 TB)(不是物理内存)。

## 8.3 报告第一个错误后继续运行

默认模式下asan只会报告第一个错误然后调用`_exit()`退出程序。

continue-after-error模式下asan不会在发现错误时退出程序。

为了启用continue-after-error模式，使用编译时flag`-fsanitize-recover=address`和运行时flag`ASAN_OPTIONS=halt_on_error=0`。

但需要注意continue-after-error模式可能不如默认模式可靠性高，且除了第一个之外的其他错误可能不准确。

## 8.4 更激进的错误检查

使用如下编译时flags和运行时flags：

```
CFLAGS += -fsanitize-address-use-after-scope
ASAN_OPTIONS=strict_string_checks=1:detect_stack_use_after_return=1:check_initialization_order=1:strict_init_order=1
```

## 8.5 静态链接

asan不能用于静态链接。

## 8.6 混用clang和gcc

**No.**

clang和gcc有完全不兼容的asan实现，不要在编译链接运行的任何过程中混用clang和gcc。

# 9. 参考文献

- https://github.com/google/sanitizers/wiki/AddressSanitizer
