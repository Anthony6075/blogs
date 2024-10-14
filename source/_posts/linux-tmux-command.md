---
title: Linux tmux命令常见用法
categories:
- computer_science
tags:
- linux
- shell
---

# 1. 简介

终端复用器，terminal multiplexer的简称。

将终端和终端里运行的命令解绑。

# 2. 常用命令

## 2.1 新建会话

- `tmux new `: 创建会话，会话名默认为编号：0,1，2...

- `tmux new -s <session-name>`: 创建会话并指定会话名

## 2.2 分离会话

`Ctrl+b d`或者`tmux detach`

## 2.3 接入会话

`tmux attach -t <session-name>`

## 2.4 切换会话

`tmux switch -t <session-name>`

## 2.5 重命名会话

`tmux rename-session -t <old-session-name> <new-session-name>`