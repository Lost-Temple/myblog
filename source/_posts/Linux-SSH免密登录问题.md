---
title: Linux-SSH免密登录问题
tags:
  - linux
  - ssh
  - key
  - 免密码
  - 免密登录
  - 远程登录
cover: 'https://s2.loli.net/2022/11/07/ORKs79jk4Hn1Xiz.png'
categories: 
  - Linux
abbrlink: 49716
date: 2022-11-02 16:09:47
---

阿里云服务器的`authorized_keys`中已经添加过我个人电脑的公钥，一开始是可以免密码登录的，后来估计不知道我个人电脑更新了啥，ssh 登录远程服务器时，提示：

```shell
Unable to negotiate with `服务器IP地址` port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
```

解决方法记录一下：

```shell
vim ~/.ssh/config # 没有就创建
```

添加以下内容，保存退出

```shell
Host xxx.xxx.xxx.xxx                          # 服务器地址
HostKeyAlgorithms +ssh-dss
PubkeyAcceptedKeyTypes ssh-rsa,ssh-dss
```

网上一般都说是添加上面的前两行就解决了，但我试了不行，只有添加了上面3行内容才解决
