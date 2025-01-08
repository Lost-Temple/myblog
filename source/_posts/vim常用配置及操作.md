---
title: vim常用配置及操作
categories:
  - Linux
tags:
  - vim
  - plugin
cover: 'https://s2.loli.net/2023/06/03/C4iRe3FtAbcuTJ6.png'
abbrlink: 44473
date: 2023-06-03 22:22:43
---

# vim 插件的安装
##  markdown-composer 插件的安装
- 在`~/.vimrc`内添加vim-markdown插件
```shell
vim ~/.vimrc
```
内容如下：
```
Plug 'euclio/vim-markdown-composer'
```

由于这个插件是由rust编写的,vim-plug 进行安装时是克隆代码下来，但不会编译，可以到插件目录下手动编译这个插件

```shell
cd ~/.vim/plugged/vim-markdown-composer
cargo build --release --no-default-features --features json-rpc
```
# 常用命令助记
- 增加文本
  - a/A(append): a 表示在当前光标的后面插入，A表示在当前行尾插入
  - i/I(insert): i 表示在当前光标的前面插入，I表示在当前行首插入
  - o/O(open a line): o 表示在当前行的下面插入一行，O表示在当前行的前面插入一行
- 删除
  - dd: 表示删除当前行
  - x/X: 向后/向前删除一个字符
  - dw(delete word): 删除光标位置后面的一个单词,可以配合数字来表示删除几个单词
  - diw(delete inner word): 当光标在某个单词内部，现在想删除当前这个单词，就使用diw
  - daw(delete around word): 和diw的区别是daw会把单词左右的空格也会删除
- 修改
  - `c`:(change)
  - `ciw`:(change inner word):这个的应用场景是把光标移到双引号中的某个字符上，然后`ciw"`,就可以把双引号中的
的字符全删除并进入插入模式
  - `ct)`:(change to `)`):这个的应用场景是把光标移动到左括号后面，然后`ct)`就可以把括号中的字符全删除并进入
到插入模式
- 查找
  - `f`:(find): 这个是在当前行查找字符或移动光标到相应的字符上，比如查找s这个字符，命令为`fs`,光标就会移动>到当前行的`s`字符上,使用`;`定位到下一个。使用数字`0`返回到行的绝对行首或`$`回到行首再进行其它字符的查找。当
然`F`为反方向的操作。
  - `/`: 在光标位置往下查找单词可以在`/`后面跟单词就会查找这个单词并高亮显示；使用`?`加单词可以往反方向查找
单词
- 光标移动
  - w(word): 移动到下一个单词
  - b(back word): 移动到前一个单词
  - 数字+G：定位行
  - 0: 移动到绝对行头
  - ^: 移动到行首
  - $: 移动到行尾
  - gg: 移动到第1行
  - G：移动到最后一行
  - ctl + o: 返回到上一个位置
  - ctl + f（forward): 往下翻页
  - ctl + u (upward): 往前翻页

# 宏定义
`normal`模式下按`q`键，再按`x`，表示使用寄存器`x`
然后就开始录制了,等录制结束，按`ESC`，再按`q`就结束录制，然后就是调用这个宏重复做刚才的事。调用方法为:
```
@x
```
表示调用x寄存器中的宏,当然可以在前面加上一个数字来表示调用次数

