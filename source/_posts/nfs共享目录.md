---
title: nfs共享目录
categories:
  - Linux
tags:
  - nfs
cover: https://s2.loli.net/2024/02/18/SofQ9nNGWbZesBy.jpg
abbrlink: 31178
date: 2024-02-18 15:17:59
---

# nfs 服务

1. 安装 NFS 服务器软件包：首先，在 Fedora 系统上安装 NFS 服务器软件包。NFS 服务器软件包通常是`nfs-utils`，你可以使用以下命令安装：
    ```Shell
    sudo dnf install nfs-utils
    
    ```

2. 配置`NFS`服务器：`NFS` 服务器的配置文件是`/etc/exports`，你需要编辑这个文件来指定要共享的目录和相关的共享选项。比如，你可以使用`vi`或者其他文本编辑器来编辑`/etc/exports`文件：

    ```Shell
    sudo vi /etc/exports
    ```

   在打开的文件中，你可以添加类似如下的行来指定共享的目录和相关选项：
    ```Shell
    /home/maodaoming/nfs_share/volume *(rw,sync,no_root_squash,no_subtree_check,insecure)
    ```
   这行表示将 /home/maodaoming/nfs_share/volume 目录共享给所有客户端,还定义了相应的权限

3. 重载 NFS 服务器配置：编辑完成 /etc/exports 文件后，你需要重新加载 NFS 服务器配置使之生效。你可以使用以下命令来重新加载：
    ```Shell
    sudo exportfs -r
    ```
   这个命令会重新加载 /etc/exports 文件中的配置，并使新的共享目录生效。

4. 启动 NFS 服务器：使用以下命令启动 NFS 服务器：
    ```Shell
    sudo systemctl start nfs-server
    ```
5. 设置开机启动
    ```Shell
    sudo systemctl enable nfs-server
    ```

# 客户端

1. 创建本地挂载点：在 macOS 上创建一个本地目录，用于挂载 NFS 共享。例如，你可以在 macOS 上创建一个名为 /mnt/nfs_share 的目录：
    ```Shell
    sudo mkdir -p /mnt/nfs_share
    ```
2. 挂载 NFS 共享：使用 mount 命令挂载 NFS 共享。你需要指定 NFS 服务器的 IP 地址（或主机名）以及共享的远程目录和本地挂载点。例如：
    ```Shell
    sudo mount -t nfs <NFS服务器IP>:/home/maodaoming/nfs_share/volume /mnt/nfs_share
    ```

3. ~~卸载挂载点~~
    ```Shell
    sudo umount /mnt/nfs_share
    ```

