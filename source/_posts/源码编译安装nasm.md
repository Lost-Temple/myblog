---
title: 源码编译安装nasm
tags:
  - nasm
categories:
  - 汇编
cover: 'https://s2.loli.net/2024/03/05/l7IdcjQfewYZRAa.jpg'
abbrlink: 38039
date: 2024-03-05 11:26:47
---

# 从源代码编译安装 Nasm

## 下载 Nasm 源代码

```shell
wget https://www.nasm.us/pub/nasm/releasebuilds/nasm-2.15.05.tar.gz
```

## 解压源代码

```shell
tar -xvzf nasm-2.15.05.tar.gz
```

## 进入源代码目录

```shell
cd nasm-2.15.05
```

## 配置编译环境

```shell
./configure
```

## 编译安装

```shell
make && sudo make install
```

## 验证 Nasm 版本

```shell
nasm --version
```
