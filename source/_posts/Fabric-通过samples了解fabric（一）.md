---
title: Fabric-通过samples了解fabric（一）
tags:
  - fabric
  - test-network
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 45946
date: 2022-11-02 17:24:40
---

# 通过fabric-samples了解fabric

需要有以下几个步骤：

- 1. 如果需要，请克隆 [hyperledger/fabric-samples](https://github.com/hyperledger/fabric-samples) 仓库
- 2. 检出适当的版本标签
- 3. 将指定版本的 Hyperledger Fabric 平台特定二进制文件和配置文件安装到 fabric-samples 下的 /bin 和 /config 目录中
- 4. 下载指定版本的 Hyperledger Fabric docker 镜像

**无梯子会比较麻烦，下面这种方法行不通**

# 方法1. 使用脚本完成

因为`curl -sSL https://bit.ly/2ysbOFE`会下载一个脚本，这个脚本如果在无梯子的情况下，是执行不成功的，可以先下载这个脚本，自行修改。

如果想要最新的生产发布版，忽略所有的版本标识符

```shell
curl -sSL https://bit.ly/2ysbOFE | bash -s
```

如果你想要一个指定的发布版本，传入一个 Fabric、Fabric-ca 和第三方 Docker 镜像的版本标识符。下边的命令显示了如何下载最新的生产发布版 - **Fabric v2.2.0** 和 **Fabric CA v1.4.7** 。

```shell
curl -sSL https://bit.ly/2ysbOFE | bash -s -- <fabric_version> <fabric-ca_version>
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.0 1.4.7
```



# 方法2. 通过fabric项目中的一个脚本

- 先`clone fabric`代码

  ```shell
  git clone https://github.com/hyperledger/fabric.git
  ```
  
- 进入到`fabric`目录下

  ```shell
  cd fabric/scripts/
  ./bootstrap.sh
  ```

  接下来无梯子还是会遇到问题,可以对bootstrap.sh中的内容进行修改，或按脚本的流程自己手动下载

# 启动测试网络

## 进入到相关目录

```shell
cd fabric-samples/test-network
```

## 删除掉先前运行的所有容器或工程

```shell
./network.sh down
```

## 启动测试网

```shell
./network.sh up
```

