---
title: Linux系统中Pycharm界面显示问题
cover: 'https://s2.loli.net/2023/01/09/7BVLPKytReJW4UX.jpg'
categories:
  - Linux
abbrlink: 17263
date: 2023-08-06 19:07:22
tags:
---

# 前言
打开`Pycharm`，激活界面中的中文显示不了，因为缺少了字体，`文泉驿正黑`，我用的是`Fedora`，记录一下`Fedora`下安装`文泉驿正黑`这个字体

# 安装`文泉驿正黑`
## 安装
```shell
sudo dnf install -y wqy-zenhei-fonts
```
> 也可以使用`Fedora`的软件包管理器安装`ttf-wqy-zenhei`字体包

## 刷新字体
```shell
fc-cache -f -v
```

## 验证
```shell
fc-list | grep "文泉驿正黑"
```

