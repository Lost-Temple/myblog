---
title: Primihub环境搭建
categories:
  - 联邦学习
tags:
  - 联邦学习
  - 多方安全计算
  - primihub
cover: 'https://s2.loli.net/2023/03/08/g4aocDiYqFANJRd.png'
abbrlink: 30445
date: 2023-03-08 11:33:56
---
# 快速开始

## 准备工作
安装`docker`和`docker-compose`，或者下载`PrimiHub`整理好的[安装包](https://primihub.oss-cn-beijing.aliyuncs.com/dev/docker20.10.tar.gz), 下载后解压执行
```shell
bash install_docker.sh
```
完成`docker`和`docker-compose`的安装。

然后下载仓库并进入到代码目录：
```shell
git clone https://github.com/primihub/primihub.git
cd primihub
```

## 启动节点
### 启动测试用的节点
使用`docker-compose`启动容器。容器包括：启动点、redis(数据集查找默认使用redis)、三个节点
```shell
docker-compose up -d
```
查看运行起来的docker容器：
```shell
docker-compose ps -a
```
如果一切顺利会看到类似如下的信息：
```shell
NAME                    COMMAND                  SERVICE                 STATUS              PORTS
primihub-node0          "/bin/bash -c './pri…"   node0                   running             0.0.0.0:6666->6666/tcp, 0.0.0.0:8050->50050/tcp
primihub-node1          "/bin/bash -c './pri…"   node1                   running             0.0.0.0:6667->6667/tcp, 0.0.0.0:8051->50051/tcp
primihub-node2          "/bin/bash -c './pri…"   node2                   running             0.0.0.0:6668->6668/tcp, 0.0.0.0:8052->50052/tcp
redis                   "docker-entrypoint.s…"   redis                   running             0.0.0.0:6379->6379/tcp
simple_bootstrap_node   "/app/simple-bootstr…"   simple_bootstrap_node   running             0.0.0.0:4001->4001/tcp
```

但是，我这里遇到坑了，node0, node1, node2 这3个容器的状态为restarting。并且是不断restarting。使用命令：
```shell
docker logs 容器ID
```
查看日志是加载mysql客户端的一个动态库文件失败。这个容器没打包好？issue总共也没有几条，更不用说搜索有没有遇到类似的问题了，可见这个项目用的人少得可怜，文档也不健全。

解决方法: 检出另外的分支，再使用 docker-compose up -d
> 测试了 feature/node_as_share_lib 是可以的，develope和master都不行

## 创建一个MPC任务
让三个节点共同执行一个多方安全计算（MPC)的逻辑回归任务

```shell
sudo docker run "--network=host" -it primihub/primihub-node:1.1.0 ./primihub-cli "--server=127.0.0.1:8050"
```

> 注：注意primihub-node的版本，要和前一章节中docker-compose启动的primihub-node版本一致

# 使用docker-compose部署

## 下载安装包，执行脚本，完成部署
```shell
curl -s https://get.primihub.com/release/latest/docker-deploy.tar.gz | tar zxf -
cd docker-deploy
bash deploy.sh
```

## 查看部署结果
```shell
# docker-compose ps -a
NAME                COMMAND                  SERVICE                 STATUS              PORTS
application1        "/bin/sh -c 'java -j…"   application1            running             
application2        "/bin/sh -c 'java -j…"   application2            running             
application3        "/bin/sh -c 'java -j…"   application3            running             
bootstrap-node      "/app/simple-bootstr…"   simple-bootstrap-node   running             4001/tcp
fusion              "/bin/sh -c 'java -j…"   fusion                  running             
gateway1            "/bin/sh -c 'java -j…"   gateway1                running             
gateway2            "/bin/sh -c 'java -j…"   gateway2                running             
gateway3            "/bin/sh -c 'java -j…"   gateway3                running             
loki                "/usr/bin/loki -conf…"   loki                    running             0.0.0.0:3100->3100/tcp, :::3100->3100/tcp
manage-web1         "/docker-entrypoint.…"   nginx1                  running             0.0.0.0:30811->80/tcp, :::30811->80/tcp
manage-web2         "/docker-entrypoint.…"   nginx2                  running             0.0.0.0:30812->80/tcp, :::30812->80/tcp
manage-web3         "/docker-entrypoint.…"   nginx3                  running             0.0.0.0:30813->80/tcp, :::30813->80/tcp
mysql               "docker-entrypoint.s…"   mysql                   running             0.0.0.0:3306->3306/tcp, :::3306->3306/tcp
nacos-server        "bin/docker-startup.…"   nacos                   running             0.0.0.0:8848->8848/tcp, 0.0.0.0:9555->9555/tcp, 0.0.0.0:9848->9848/tcp, :::8848->8848/tcp, :::9555->9555/tcp, :::9848->9848/tcp
primihub-node0      "/bin/bash -c './pri…"   node0                   running             0.0.0.0:6666->6666/tcp, 0.0.0.0:50050->50050/tcp, :::6666->6666/tcp, :::50050->50050/tcp
primihub-node1      "/bin/bash -c './pri…"   node1                   running             0.0.0.0:6667->6667/tcp, 0.0.0.0:50051->50051/tcp, :::6667->6667/tcp, :::50051->50051/tcp
primihub-node2      "/bin/bash -c './pri…"   node2                   running             0.0.0.0:6668->6668/tcp, 0.0.0.0:50052->50052/tcp, :::6668->6668/tcp, :::50052->50052/tcp
rabbitmq1           "docker-entrypoint.s…"   rabbitmq1               running             25672/tcp
rabbitmq2           "docker-entrypoint.s…"   rabbitmq2               running             25672/tcp
rabbitmq3           "docker-entrypoint.s…"   rabbitmq3               running             25672/tcp
redis               "docker-entrypoint.s…"   redis                   running             6379/tcp
```

## 使用说明
docker-compose.yaml 文件中的nginx1、nginx2、nginx3 模拟 3 个机构的管理后台，启动完成后在浏览器分别访问

http://机器IP:30811

http://机器IP:30812

http://机器IP:30813

默认用户密码都是 admin / 123456

具体的联邦建模、隐私求交、匿踪查询等功能的操作步骤请参考 [快速试用管理平台](https://docs.primihub.com/docs/quick-start-platform)


# primihub代码编译

## Mac Silicon M1
- 安装Xcode
- 安装Bazel 5.0.0，最好是直接brew install bazelisk
- 安装CMake，这个最坑，缺了这个，在这里卡了好久
- 编写一个shell脚本，用来执行编译命令:
    ```shell
    #!/bin/bash

    ./pre_build.sh
    bazel build --config=darwin_arm64 --config=macos  :node :cli :opt_paillier_c2py :linkcontext
    ```

# 节点运行
在编译成功后，在源码目录下生成了可执行文件，路径为：
```
./bazel_bin/node
```
可使用下面命令运行node

```shell
./bazel-bin/node --node_id=node0 --service_port=50050 --config=./config/node0.yaml
```

```shell
./bazel-bin/node --node_id=node1 --service_port=50051 --config=./config/node1.yaml
```

```shell
./bazel-bin/node --node_id=node2 --service_port=50052 --config=./config/node2.yaml
```
