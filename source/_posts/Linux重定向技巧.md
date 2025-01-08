---
title: Linux重定向技巧
date: 2025-01-07 18:03:28
tags:
  - linux
  - 重定向
cover: 'https://s2.loli.net/2022/11/07/ORKs79jk4Hn1Xiz.png'
categories: 
  - Linux
---

# linux输出重定向

```shell
command > file 2>&1
command >> file 2>&1
```

放在`>`后面的`&`，表示重定向的目标不是一个**文件**，而是一个**文件描述符**，内置的文件描述符如下:
```
1 => stdout
2 => stderr
0 => stdin
```

换言之 `2>1` 代表将**stderr**重定向到当前路径下文件名为**1**的**regular file**中，而`2>&1`代表将**stderr**重定向到**文件描述符**为**1**的文件(即`/dev/stdout`)中，这个文件就是**stdout**在**file system**中的映射
而**&>file**是一种特殊的用法，也可以写成**>&file**，二者的意思完全相同，都等价于

```
>file 2>&1
```

> 此处`&>`或者`>&`视作整体，分开没有单独的含义

**顺序问题：**

```shell
find /etc -name .bashrc > list 2>&1
# 为什么不能调下顺序,比如这样
find /etc -name .bashrc 2>&1 > list
```

第一种：

```shell
xxx > list 2>&1
```

- **含义：**

​	`> list`：将**标准输出**（文件描述符 1）重定向到文件 list。

​	`2>&1`：将**标准错误**（文件描述符 2）重定向到与标准输出相同的位置。

- **执行顺序：**

​	1. > list 首先将标准输出指向文件 list。

​	2. 2>&1 将标准错误指向与标准输出相同的地方（即文件 list）。

- **效果：**

​	**标准输出和标准错误都被重定向到文件** list **中**。

​	这是正确的用法。

第二种：

```shell
xxx 2>&1 > list
```

- **含义：**

​	`2>&1`：将**标准错误**（文件描述符 2）重定向到当前的标准输出。

​	`> list`：将**标准输出**（文件描述符 1）重定向到文件 list。

- **执行顺序：**

​	1. 2>&1 首先将标准错误指向当前标准输出的位置。

​	2. 然后，> list 将标准输出重定向到文件 list，但此时标准错误仍然指向原始的标准输出（可能是终端）。

- **效果：**

​	**标准输出被写入文件** list。

​	**标准错误仍然输出到终端**（因为它指向的是原始标准输出，而不是文件 list）。

​	导致无法实现标准输出和标准错误的统一重定向。

# 输入重定向

```shell
wc < test.txt
```

把test.txt的内容作为`wc`的输入，效果和以下命令类似：

```shell
cat test.txt | wc
```

把test.txt的内容通过管道传输给`wc`

# dev/null

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：

```shell
command > /dev/null
```

/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。

如果希望屏蔽 `stdout `和 `stderr`，可以这样写：

```shell
command > /dev/null 2>&1
```

# 重定向深入

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
- 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
- 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

默认情况下，command > file 将 stdout 重定向到 file，command < file 将stdin 重定向到 file。

如果希望 stderr 重定向到 file，可以这样写:

```shell
command 2 > file
```

如果希望 stderr 追加到 file 文件末尾，可以这样写:

```shell
command 2 >> file
```

如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：

```shell
command > file 2>&1
# 或者
command >> file 2>&1
```

如果希望对 stdin 和 stdout 都重定向，可以这样写：

```shell
command < file1 >file2
```

command 命令将 stdin 重定向到 file1，将 stdout 重定向到 file2。

注意：为什么需要将标准错误重定向到标准输出，因为标准错误没有缓冲区，而stdout有。

| 命令            | 说明                                               |
| --------------- | -------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |
