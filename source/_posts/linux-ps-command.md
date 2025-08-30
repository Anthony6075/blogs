---
title: Linux ps常用命令
date: 2024-12-03 10:57:38
categories:
- computer_science
tags:
- linux
- shell
---

# 1. 查看所有进程

```bash
$ ps aux
```

# 2. 查看进程启动时间

```bash
$ ps -eo pid,lstart,cmd
```