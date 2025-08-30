---
title: Linux vim常用命令
date: 2024-05-08 08:59:02
categories:
- computer_science
tags:
- linux
- shell
---

# 1. 查找

## 1.1 向后查找

命令为`/<search_word>`然后回车，使用`n`定位到下一次出现，使用`N`定位到上一次出现

## 1.2 向前查找

使用命令`?<search_word>`

## 1.3 case-insensitive

查找默认是case-sensitive的，若想insensitive，在`<search_word>`前或者后添加`\c`

例如`/<search_word>\c`

# 2. 替换

## 2.1 全文替换

命令为`:%s/<search_word>/<replace_word>/g`

将全文中所有的`<search_word>`替换为`<replace_word>`，这是case-sensitive的

若想case-insensitive，给上述命令添加`i`选项

## 2.2 每次替换时询问

若想在每次替换实际发生时询问是否确认，给上述命令添加`c`选项(c表示confirmation)，即

`:%s/<search_word>/<replace_word>/gc`

在prompt出现时的回答包括：

1. `y`，表示yes，即这次替换

1. `n`，表示no，即这次不替换

1. `a`，表示all，即从此往后的出现都直接替换并使命令退出

1. `q`，表示quit，这次不替换并使命令退出

1. 以及其他
