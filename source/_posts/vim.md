---
title: 【Linux系列教程】vim编辑器
date: 2023-02-21 14:31:35.416
updated: 2023-02-24 20:14:58.843
categories: 
- 技术
- 虚拟机
- 笔记
- centos
- linux教程
tags: 
- centos
- 虚拟机
- linux基础
- linux
- vim
- vi
---

# 【Linux系列教程】vim编辑器

## 一、介绍

- 功能：修改文件内容
- 特性：模式化编辑器
- 三个工作模式：命令模式、插入模式、末行模式

```
使用方法

vim 文件名称
```

## 二、模式切换

1.  命令模式  ----> 插入模式
	1. a, i, o, O
1. 插入模式 ----> 命令模式
	1. ESC 
1. 命令模式 ----> 末行模式
	1. 冒号  :

## 三、末行模式

1. 保存退出
	1. :wq
1. 不保存
	1. :q!
1. 显示行号
	1. :set nu

## 四、命令模式

### 1.快速移动光标 

- h左    j下    k 上  l 右
- $	移动到行尾
- ^	移动到行首
- 68gg		跳转到68行
- shift + g 跳转到最后一行
- gg		跳转到第一行
- . 	重复上一次操作

### 2.删除字符

- x		删除单个字符
- dw		删除词
- dd		删除整行
- 67dd 删除(剪切)第67行
- d$		删除到行尾
- d^		删除到行首
- dG		删除到文件最后一行

### 3.复制粘贴剪切

- yy		复制整行
- p		粘贴
- dd	剪切

### 4.撤销

- u	撤销

### 5.搜索

1. /内容
	1. n	向下查找 
	1. N	向上查找 


## 四、拓展

### 1.更多键位图

![vim](https://www.wangshengjj.work/upload/2023/02/vim.png)

### 2.vim的使用练习

```
[root@localhost ~]# vimtutor
```

### 3.配置vim编辑器的个人使用习惯

**比如，打开某个文件后，默认显示行号，进行TAB缩进时，把Linux默认的站位8个改成4个**

```
[root@localhost ~]# vim /etc/vimrc

if v:lang =~ "utf8$" || v:lang =~ "UTF-8$"
  2    set fileencodings=ucs-bom,utf-8,latin1
  3 endif
  4 
  5 set nocompatible    " Use Vim defaults (much better!)
  6 set bs=indent,eol,start     " allow backspacing over everything in inse    rt mode
  7 "set ai         " always set autoindenting on
  8 "set backup     " keep a backup file
  9 set viminfo='20,\"50    " read/write a .viminfo file, don't store more
 10             " than 50 lines of registers
 11 set history=50      " keep 50 lines of command line history
 12 set ruler       " show the cursor position all the time
 13 set nu	#添加此内容，默认打开某个文件时，自动显示行号
 14 set tabstop=4	#添加此内容，默认缩进4个空格
```
