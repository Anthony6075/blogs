---
title: Bash job control
date: 2024-06-03 16:09:06
categories:
- computer_science
tags:
- linux
- shell
---

# 1. job control是什么

job control指**暂停执行进程**并稍后**继续执行**它的能力。

bash给每个pipeline关联一个job。

# 2. 前台进程和后台进程

前台(foreground)进程指，与当前终端进程的进程组id(process group id)相同的进程； 反之为后台(background)进程。

区别包括：

1. 前台进程接受**键盘生成的信号**(如ctrl+c生成SIGINT)，后台进程不会接受。

1. 只有前台进程被允许**读取终端**，后台进程如果尝试读终端会被发送SIGTTIN信号，它会暂停(suspend)该进程。

# 3. job spec

1. `%<n>`指第n个job

1. `%`和`%%`和`%+`指当前job(即最后一个从foreground暂停或从background启动的job)

1. `%-`指前一个job

# 4. 使用

## 4.1 `Ctrl+Z`(suspend字符)

暂停进程，同时控制权返回给bash。

## 4.2 `Ctrl+Y`(delayed suspend字符)

在进程尝试**读终端**时，暂停进程并把控制权返回给bash。

## 4.3 `bg`命令

用法为`bg [<jobspec>...]`

使指定的job在后台恢复执行(仿佛它们启动时在命令行加了`&`)。

如果不指定jobspec，使用当前job。

使用`<jobspec> &`也一样。

## 4.4 `fg`命令

用法为`fg [<jobspec>]`

使指定的job在前台恢复执行，它也成为**当前job**。

如果不指定jobspec，使用当前job。

仅使用`<jobspec>`不加fg也一样。

## 4.5 `jobs`命令

用法为`jobs [-lrs] [<jobspec>]`

用于列举jobs。带`<jobspec>`则只列举它，否则列举所有jobs。

标记`+`的为当前job，标记`-`的为前一个job。

- `-l` 同时列出进程ID

- `-r` 只列出running的jobs

- `-s` 只列出stopped的jobs

## 4.6 `kill`命令

用法为`kill [-sigspec] <jobspec>|<pid>`

给`<jobspec>`或者`<pid>`标识的进程发送信号`sigspec`。

如果`-sigspec`未指定则默认发送`SIGTERM`。

`kill -l|-L`可列出信号名和数字的对应关系。