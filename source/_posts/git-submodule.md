---
title: git submodule命令用法
categories:
- computer_science
tags:
- git
---

# 1. 含义

submodule指向子模块仓库一个特定的commit，也可以追踪子模块仓库一个特定的分支

# 2. 常用命令

## 2.1 添加子模块

```bash
git submodule add [-b branch] <repository> [<path>]
```

添加`<repository>`的branch分支到`<path>`目录

## 2.2 clone主仓库

```bash
git clone --recursive <parent-repository>
```

在克隆主仓库时同时克隆其中包含的子模块

## 2.3 下载主仓库的所有子模块内容

```bash
git submodule update --init --recursive
```

## 2.4 更新到子模块远程仓库的状态

```bash
git submodule update --remote
```

这个命令会改变submodule指向的子模块仓库中的具体commit

## 2.5 修改submodule指向的commit

```bash
cd <path-to-submodule>

git checkout <some-commit>

cd <path-to-parent-repository>

git add <path-to-submodule>

git commit
```

去子模块目录修改commit，然后在主仓库把这个更改commit即可

## 2.6 删除子模块

使用下述三条命令

```bash
git rm <path-to-submodule>

rm -rf .git/modules/<path-to-submodule>

git config --remove-section submodule.<path-to-submodule>
```