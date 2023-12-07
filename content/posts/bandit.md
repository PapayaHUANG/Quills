---
title: 'Bandit 学习笔记'
date: 2023-12-05T16:29:19+08:00
draft: false
summary: 'Bandit 闯关日记'
tags: [unix, shell]
categories: [studyNotes]
---

## 学习网址

- 游戏地址
  https://overthewire.org/wargames/bandit/
- Linux 命令查询地址
  https://man7.org/linux/man-pages/index.html

## Level 0 - Level 13

### 涉及知识点

#### 1. 什么是 `SSH`

SSH 的全称是 Secure Shell Protocol（安全外壳协议），是一种加密的网络传输协议。应用场景是远程登录和执行命令行。
SSH 使用非对称加密来实现身份验证。

#### 2. 特殊命名文件的打开

- 当文件名是 `-` 时

  可以使用 `cat < -` 或者 `./-`, 前者是把标准输入重定向到文件名为 `-`的文件， 后者是给出完整路径

  特殊原因：因为`-` 可以被认为是**标准输入**

- 当文件名中有空格

  可以使用引号框住文件名，或者用 `\`取消掉空格

#### 3. 搜索命令

`find` 是根据**文件的属性**来进行查找；`grep` 是根据**文件的内容**进行查找。

#### 4. 管道和重定向 Piping and Redirection

每一个在命令行运行的程序都有 3 种工作流：

- STDIN
- STDOUT
- STDERR

  [参考资料： Ryans Linux Tutorial](https://ryanstutorials.net/linuxtutorial/piping.php)

#### 5 解码

- base64 解码： `base64 -d`
- translate: `tr`

  ROT13: `tr 'A-Za-z' 'N-ZA-Mn-za-m'`

#### 6 压缩方式

- tar 并没有压缩
- gzip GNU zip
- bzip2
- zip 主要使用于 windows

  [参考资料：Archive under Linux](<https://www.thomas-krenn.com/en/wiki/Archive_under_Linux_(tar,_gz,_bz2,_zip)>)
