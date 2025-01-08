---
title: vim常用插件的使用备忘录
categories:
  - Linux
tags:
  - vim
  - plugin
cover: 'https://s2.loli.net/2023/06/03/C4iRe3FtAbcuTJ6.png'
abbrlink: 9663
date: 2023-06-16 20:50:11
---


# `vim-surround` 的用法
- `cs'"`用双引号替换单引号
- `cs"'`用单引号替换双引号
- `ysiw<q>`给当前单词加上标签`<q>`
- `cst"`用双引号替换当前的标签
- `ds'`删除单引号
- `ysiw]`在当前单词外加上方括号
- `yss]`在当前句子处加上方括号  

# vim编辑`markdown`在浏览器中预览效果
- 安装`coc-nvim`插件
- 使用`coc-nvim`插件安装`coc-markdown`插件
- 使用`:CocCommand markdown-preview-enhanced.openPreview`命令在浏览器中打开预览

# `coc-vim`的使用
- 可以先安装一个`coc-marketplace`插件:`:CocInstall coc-marketplace`
- `CocList marketplace`列出所有可用的插件
- 在选中的插件上可以使用`tab`来进行`install`,`uninstall`,`homepage`等动作

# `vim-easy-align`的使用

```
  Lorem   = ipsum
  dolor   = sit
   amet  += consectetur == adipiscing
   elit  -= sed           != do
eiusmod   = tempor        = incididunt
     ut &&= labore
```
- normal模式下：`gaip`
  - `=`Around the 1st occurrentces
  - `2=`Around the 2nd occurrences
  - `*=`Around all occurrences
  - `**=`Left/Right alternating alignment around all occurrences
  - `<Enter>`Switching between left/right/center alignment modes

- `visual line` 模式: `vip`
  - `ga`进入`easyalign`模式
  - `<Enter>` 切换对齐模式
  - 输入`对齐目标字符`, 再`<Space>` 完成对齐操作

# `nerdtree`的使用
- `j`, `k` 在目录下上移动
- `o`展开目录或打开文件，焦点会跑到右侧文件视图中
- `ctrl + w + h`可以让焦点从文件中回到左侧目录视图中
- `:NERDTreeToggle`命令是`NERDTree`打开或关闭的开关,在`.vimrc`中设置快捷键把它用`F3`来控制
    ```shell
    " 设置NerdTree
    map <F3> :NERDTreeMirror<CR>
    map <F3> :NERDTreeToggle<CR>
    ```
# `TagBar`的使用
- `:TagbarToggle`
