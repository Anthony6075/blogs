---
title: Linux GDB TUI(Text User Interface)
categories:
- computer_science
- gdb
tags:
- linux
- gdb
- shell
---

# 1. GDB tui常用命令

## 1.1 启动和退出tui模式

- 分别是`tui enable`(缩写为`tui e`)和`tui disable`(缩写为`tui d`)命令

- 或者使用快捷键ctrl+x+a，按第一次进入，第二次退出

## 1.2 ctrl+L 刷新屏幕

在程序使用标准输出和标准错误打印时屏幕可能乱掉，可以使用ctrl+L进行刷新

ctrl+L不会被记在命令历史里，下一次ENTER不会重复ctrl+L

## 1.3 `info win`命令

列出当前显示的所有窗口

## 1.4 `focus`命令

缩写为`fs`，改变焦点到不同窗口，使用方式`focus [WINDOW-NAME | next | prev]`

## 1.5 查看文档

使用`help text-user-interface`查看tui相关所有命令

# 2. CGDB

CGDB是GDB的一个控制台前端(Console Frontend)，它还是使用GDB进行实际调试

GDB的tui模式不支持源码窗口和命令窗口垂直分屏，因此使用CGDB

## 2.1 使用

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

## 2.2 资源

- [官网](https://cgdb.github.io/)

- [文档](https://cgdb.github.io/docs/cgdb-split.html)

## 2.3 注意

- 目前最新版为0.8.0，没有发正式版
