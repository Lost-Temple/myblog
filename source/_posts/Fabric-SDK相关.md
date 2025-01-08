---
title: Fabric SDK相关
tags:
  - 通道
  - 链码
  - Peer
  - Orderer
  - gRPC
categories:
  - 区块链
  - Fabric
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
abbrlink: 21718
date: 2022-12-05 16:28:37
---

# Fabric SDK 介绍
`Fabric`的`Peer`节点和`Orderer`节点都提供了基于`gRPC`协议的接口，用于和`Peer`节点与`Orderer`节点进行命令/数据交互。为了简化开发，为开发人员开发应用程序提供操作`Fabric`区块链网络的API，Fabric官方提供了多种语言版本的SDK。
Fabric提供了多种语言版本的SDK：
- [fabric-sdk-java](https://github.com/hyperledger/fabric-sdk-java)
- [fabric-sdk-go](https://github.com/hyperledger/fabric-sdk-go)
- [fabric-sdk-node](https://github.com/hyperledger/fabric-sdk-node)
- [fabric-sdk-py](https://github.com/hyperledger/fabric-sdk-py)

`Fabric`区块链应用可以通过SDK访问`Fabric`区块链网络中的多种资源，包括账本、交易、链码、事件、权限管理等。应用程序代表用户与`Fabric`区块链网络进行交互，`Fabric SDK API`提供了如下功能：
- 创建通道
- 将`Peer`节点加入通道
- 在`Peer`节点安装链码
- 在通道实例化链码
- 通过链码调用交易
- 查询交易或区块的账本

# Fabric SDK 安装
```shell
go get -u github.com/hyperledger/fabric-sdk-go
```

# Fabric Go SDK 源码结构
```
total 624
-rw-r--r--   1 mao  staff   170K Aug 15 14:45 CHANGELOG.md
-rw-r--r--   1 mao  staff   108B Aug 15 14:45 CODEOWNERS
-rw-r--r--   1 mao  staff   577B Aug 15 14:45 CODE_OF_CONDUCT.md
-rw-r--r--   1 mao  staff   661B Aug 15 14:45 CONTRIBUTING.md
-rw-r--r--   1 mao  staff    11K Aug 15 14:45 LICENSE
-rw-r--r--   1 mao  staff   910B Nov  2 08:36 MAINTAINERS.md
-rw-r--r--   1 mao  staff    33K Aug 15 14:45 Makefile
-rw-r--r--   1 mao  staff   7.3K Aug 15 14:45 README.md
-rw-r--r--   1 mao  staff   1.0K Aug 15 14:45 SECURITY.md
drwxr-xr-x   4 mao  staff   128B Aug 15 14:45 ci
-rw-r--r--   1 mao  staff   143B Aug 15 14:45 ci.properties
-rw-r--r--   1 mao  staff   2.3K Aug 15 14:45 doc.go
-rw-r--r--   1 mao  staff   1.2K Aug 15 14:45 go.mod
-rw-r--r--   1 mao  staff    45K Aug 15 14:45 go.sum
-rwxr-xr-x   1 mao  staff   1.6K Aug 15 14:45 golangci.yml
drwxr-xr-x   3 mao  staff    96B Aug 15 14:45 internal
drwxr-xr-x  11 mao  staff   352B Aug 15 14:45 pkg
drwxr-xr-x   5 mao  staff   160B Aug 15 14:45 scripts
drwxr-xr-x   7 mao  staff   224B Aug 15 14:45 test
drwxr-xr-x   3 mao  staff    96B Aug 15 14:45 third_party
```

- `pkg/fabsdk`: Fabric SDK的主要包、允许基于配置创建上下文。上下文由客户端软件包使用。
- `pkg/client/channel`: 提供通道交易相关功能
- `pkg/client/event`: 提供通道事件相关功能
- `pkg/client/ledger`: 启用对通道底层账本的查询相关功能
- `pkg/client/resmgmt`: 提供资源管理功能，例如安装链码
- `pkg/client/msp`: 启用身份管理相关功能

# Fabric SDK功能模块
## API
对于应用开发者来说，插件化的API可以支持SDK提供的关键接口的可选实现。对于每个接口，都有内置的默认实现，也可以灵活自定义。

## fabric-client
`fabric-client`模块提供API与基于`Hypreledger Fabric`区块链网络的核心组件（即`peer`，`order`和`事件流`）进行交互，主要功能如下：
- 创建`channel`
- 请求`Peer`节点加入通道
- 在`Peer`节点中安装链码
- 在通道中实例化链码
- 通过调用链码来调用事务
- 多种查询
- 监听事件

## fabric-ca-client
`fabric-ca-client`模块提供与可选组件`fabric-ca`进行交互的API，`fabric-ca`提供成员管理服务。`fabric-ca-client`模块主要功能如下：
- 注册新用户
- 注册用户以获得由`Fabric CA`签名的注册证书
- 通过注册ID撤销现有用户或撤消证书
- 可定制的持久化存储