---
title: git stash命令用法
categories:
- computer_science
- git
tags:
- git
---

# 1. stash特定文件

```bash
git stash push [-m message] <path>...
```

# 2. 展示stash内容

```bash
git stash show [-p] <index>
```

`<index>`为stash索引号数字，不带`-p`选项只显示变动文件名，带上`-p`显示变动内容