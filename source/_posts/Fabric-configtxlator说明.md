---
title: Fabric-configtxlator说明
tags:
  - 二进制
  - JSON
  - 转换
categories:
  - 区块链
  - Fabric
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
abbrlink: 8253
date: 2023-01-28 14:44:35
---

# 作用
因为无论配置交易文件`.tx`和初始区块文件`.block`都是二进制格式，用户无法直接编辑。`configtxlator`工具将这些配置文件在二进制格式和方便阅读的`json`格式之间进行转换。开发`configtxlator`工具是为了支持独立于`SDK`来进行重新配置。`configtxlator`工具被设计为提供一个API让任意SDK的用户都能够与之交互来更新配置。工具的名称是`configtx`和`translator`的拼接，意在传达该工具简单地在不同的等效数据之间进行转换。它不产生配置，也不提交或撤回配置。它不修改配置本身，只是简单地提供一些配置格式的不同的映射展现。

本地命令行可使用命令：
- 编码(proto_encode)
- 解码(proto_decode)
- 对比修改结构(compute_update)
- 版本信息(version)

远程`RESTful`路由:
- 编码(proto_encode)
- 解码(proto_decode)
- 对比修改结构(compute_update)

# 使用步骤
- 使用SDK获取最新的配置
- 然后使用`configtxlator`工具将二进制文件转换为可读的配置文件
- 编辑可读的配置文件
- 使用`configtxlator`工具计算更新的配置与原有配置的差异
- 使用SDK提交配置以及签名
---
## 本地使用
在`cli`中动态增加组织的时候，我们在命令行下直接使用`configtxlator proto_encode`和`configtxlator proto_decode`这种方式。

`proto_encode`和`proto_decode`两个命令所拥有的参数是一样的，都需要以下几个参数：
- `type`: 消息结构类型
- `input`: 输入参数，encode时值为json格式，decode时值为proto格式的字节数组。
- `output`: 输出结果，和输入相对，输出的是json或者proto格式。

type包含以下几种:
| -- | -- |
| type |  description |
| -- | -- |
| common.Block | 区块结构 |
| common.Envelope | 带有效载荷和数字签名的数字信封，区块的数据部分就是序列化后的数字信封 |
| common.ConfigEnvelope | 包含链配置的数字信封，内容包含ConfigUpdateEnvelope |
| common.ConfigUpdateEnvelope | 提交给排序节点的配置数字信封 |
| common.Config | ConfigEnvelope的配置部分 |
| common.ConfigUpdate | ConfigUpdateEnvelope的一部分 |

`compute_update`计算修改量，命令有以下几个参数
- `original`: 原始proto
- `updated`: 修改后的proto
- `channel_id`: 通道id
- `output`: 对比计算后的目标配置proto

举例：将当前`channel`的块消息的配置`decode`为`json`文件
```shell
configtxlator proto_decode --input orderer.block --type common.Block
```
> 这里所使用的 orderer.block 文件请参考 [Fabric开发模式运行](http://www.blockchainof.com/2023/01/10/Fabric%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E8%BF%90%E8%A1%8C/) 2.4中所生成的创始块对应的文件。只要是区块数据，都可以使用这个命令进行解析。

## 远程调用RESTful接口
和本地调用类似，不详细介绍，直接举例吧：
```shell
curl -X POST --data-binary @orderer.block "${CONFIGTXLATOR_URL}/protolator/decode/common.Block"
```