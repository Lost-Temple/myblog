---
title: Hexo服务器搭建
tags:
  - Hexo
  - git
  - nginx
  - 免密码登录
cover: 'https://s2.loli.net/2022/10/14/txqQJYhyZmMe3Xp.png'
categories: 
  - 环境搭建-Hexo
abbrlink: 5747
date: 2022-10-12 22:09:36
---

# 第一部分：服务端操作

## 1. 安装git和nginx

```shell
yum install -y nginx git
```

## 2. 添加用户并做免密码登录

### 2.1 服务器端root用户免密登录

```shell
vim ~/.ssh/authorized_keys    #root用户的免密码登录，将本地机器的ssh公钥添加进去
```

### 2.2 服务器端添加git用户

```shell
useradd git
passwd git

# 给git用户配置sudo权限
chmod 740 /etc/sudoers
vim /etc/sudoers
# 找到root ALL=(ALL) ALL，在它下方加入一行
git ALL=(ALL) ALL

chmod 400 /etc/sudoers
```

### 2.3 git用户免密登录

```shell
su - git
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorzied_keys
chmod 700 ~/.ssh
vim ~/.ssh/authorized_keys   #git用户的免密码登录，将本地机器的ssh公钥添加进去
```

## 3. 创建仓库并使用git-hooks实现自动部署

```shell
sudo mkdir -p /var/repo #新建目录，这是git仓库的位置
sudo mkdir -p /var/www/hexo #站点根目录
cd /var/repo
sudo git init --bare blog.git #创建一个名叫blog的仓库
```

## 4. 创建post-update脚本

```shell
sudo vim /var/repo/blog.git/hooks/post-update
```

脚本内容如下：

```shell
#!/bin/bash
git --work-tree=/var/www/hexo --git-dir=/var/repo/blog.git checkout -f
```

添加权限

```shell
cd /var/repo/blog.git/hooks/
sudo chown -R git:git /var/repo/
sudo chown -R git:git /var/www/hexo
sudo chmod +x post-update  #赋予其可执行权限
```

## 5.配置nginx

```shell
cd /etc/nginx/conf.d
vim blog.conf
```

`blog.conf`的内容如下：

```shell
server {
    listen    80 default_server;
    listen    [::] default_server;
    server_name    www.blockchainof.com;
    root    /var/www/hexo;
}
```

检查Nginx配置语法并重启nginx:

```shell
nginx -t # 检查语法
nginx -s reload # 重新加载
```

# 第二部分：本地配置（Mac）

## 1. 安装Hexo及其插件

```shell
sudo npm install hexo-cli hexo-server hexo-deployer-git -g
```

## 2. 本地初始化博客站点

```shell
hexo init ~/blog
npm install hexo-deployer-git --save
```

## 3. 本地Hexo配置

```shell
# 修改Hexo的deploy配置
cd blog
vim _config.yml

# 找到deploy配置部分
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: root@xxx.xx.xxx.xxx:/var/repo/blog.git # IP填写自己服务器的IP即可
  branch: master
```

## 4. 将本地Hexo部署到远程服务器

```shell
# 清除缓存
hexo clean

# 生成静态页面
hexo generate

# 将本地静态页面目录部署到云服务器
hexo delopy
```

可以将上面的3条指令放在一个shell脚本内，方便部署

```shell
#!/bin/sh

hexo clean
hexo generate
hexo deploy
```

