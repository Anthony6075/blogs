---
title: Linux生成静态库
date: 2024-05-05 15:59:28
categories:
- computer_science
- linker
tags:
- linux
- shell
- linker
---

# 1. 使用`ar -M [ < merge.script ]`命令

写一个脚本`merge.script`，将脚本重定向给`ar -M`命令的stdin。

使用`create`传入输出的静态库文件名，`addlib`传入输入的静态库，`addmod`传入输入的目标文件，最后调用`save`和`end`。

```
create <libmerged.a>
addlib <libfirst.a>
addlib <libsecond.a>
...
addmod <object_first.o>
addmod <object_second.o>
...
save
end
```

# 2. 注意排列顺序

把必须包括其所有符号的输入库放在前面，因为在后面库中包含重复符号名的时候**可能**直接丢弃以前面为准。

# 3. 参考

- [stackoverflow](https://stackoverflow.com/a/23621751/18781047)

- [文档](https://sourceware.org/binutils/docs/binutils/ar-scripts.html)