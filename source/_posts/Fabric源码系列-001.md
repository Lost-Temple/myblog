---
title: Fabric源码系列-001
tags:
  - fabric
  - 源码
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 32507
date: 2022-11-11 16:49:03
---

# 源码总目录结构

|  目录名   | 说明  |
|  ----  | ----  |
| bccsp | 全称是区块链密码服务提供者，用来提供区块链相关的算法标准和他们的实现 |
| ccaas_builder  | 编译相关 |
| ci  | 持续集成(CI)相关配置和脚本 |
| cmd | 命令行操作相关入口代码 | 
| common  | 一些公共库（错误处理、日志处理、账本存储、策略以及各种工具等等） | 
| core | 核心库，组件的核心逻辑，针对每一个组件都有一个子目录（chaincode:与智能合约相关);(endorser:与背书节点相关）|
| discovery | 服务发现模块相关代码 |
| docs | 文档相关 |
| gossip | 组织内部节点数据同步的通信协议，最终一致性算法，用于组织内部数据同步 |
| images | Docker镜像打包，Docker镜像都是通过这个目录下的配置文件生成的 |
| integration | 待迁移代码 |
| internal | 内部代码，被cmd包等调用 |
| msp | 成员服务管理（member service provider），在Fabric网络中会为每一个成员提供相应的证书，msp模块就是读取这些证书并做一些相应的处理 |
| orderer | 排序节点的入口，用于消息的订阅与分发处理 |
| pkg | 重写或实现了一些golang原生接口 |
| protoutil | Proposal提案相关的工具包 |
| release_notes | 发布笔记 |
| sampleconfig | 配置示例 |
| scripts | 脚本，包含启动脚本、检查脚本等 |
| swagger | swagger文档生成配置 |
| tools | 工具包，目前还是空的（release-2.4分支）|
| vagrant | 包含了用 Vagrant 建立一个简单的开发环境所必需的脚本 |
| vendor | 存放Go中使用的第三方包 |


# 模块分类
## 核心模块

提供核心功能服务的代码，包括`core`、`orderer`两个代码包，这两个代码包涵盖了Orderder排序节点与Peer节点（包含Endorser背书节点与Committer记帐节点）的核心代码

## 公共模块

为核心模块和其它模块提供基础支持服务，包括`bccsp` `common` `gossip` `msp` `protoutil`等目录代码。其中`gossip`消息模块为`peer`节点提供安全、可靠、可扩展的P2P数据分发协义。`commom`公共功能模块包括帐本数据存储模块、安全服务模块、通道配置等，为其它模块提供底层存储机制、安全机制、异步通信机制等。

## 辅助模块

为其它模块提供辅助工具、运行环境、测试用例、文档等，包括`ccaas_builder` `ci` `docs` `release_notes` `swagger`等

