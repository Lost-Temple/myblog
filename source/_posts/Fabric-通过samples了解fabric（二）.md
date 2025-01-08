---
title: Fabric-通过samples了解fabric（二）
tags:
  - up
  - down
  - creatChannel
  - up createChannel
  - deployCC
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 8960
date: 2022-11-04 14:42:56
---

# network.sh 脚本命令

```shell
./network.sh help
Using docker and docker-compose
Usage:
  network.sh <Mode> [Flags]
    Modes:
      up - Bring up Fabric orderer and peer nodes. No channel is created
      up createChannel - Bring up fabric network with one channel
      createChannel - Create and join a channel after the network is created
      deployCC - Deploy a chaincode to a channel (defaults to asset-transfer-basic)
      down - Bring down the network

    Flags:
    Used with network.sh up, network.sh createChannel:
    -ca <use CAs> -  Use Certificate Authorities to generate network crypto material
    -c <channel name> - Name of channel to create (defaults to "mychannel")
    -s <dbtype> - Peer state database to deploy: goleveldb (default) or couchdb
    -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
    -d <delay> - CLI delays for a certain number of seconds (defaults to 3)
    -verbose - Verbose mode

    Used with network.sh deployCC
    -c <channel name> - Name of channel to deploy chaincode to
    -ccn <name> - Chaincode name.
    -ccl <language> - Programming language of the chaincode to deploy: go, java, javascript, typescript
    -ccv <version>  - Chaincode version. 1.0 (default), v2, version3.x, etc
    -ccs <sequence>  - Chaincode definition sequence. Must be an integer, 1 (default), 2, 3, etc
    -ccp <path>  - File path to the chaincode.
    -ccep <policy>  - (Optional) Chaincode endorsement policy using signature policy syntax. The default policy requires an endorsement from Org1 and Org2
    -cccg <collection-config>  - (Optional) File path to private data collections configuration file
    -cci <fcn name>  - (Optional) Name of chaincode initialization function. When a function is provided, the execution of init will be requested and the function will be invoked.

    -h - Print this message

 Possible Mode and flag combinations
   up -ca -r -d -s -verbose
   up createChannel -ca -c -r -d -s -verbose
   createChannel -c -r -d -verbose
   deployCC -ccn -ccl -ccv -ccs -ccp -cci -r -d -verbose

 Examples:
   network.sh up createChannel -ca -c mychannel -s couchdb
   network.sh createChannel -c channelName
   network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
   network.sh deployCC -ccn mychaincode -ccp ./user/mychaincode -ccv 1 -ccl javascript
```

## 清除测试网中的所有容器

```shell
./network.sh down
```

## 启动测试网

```shell
./network.sh up
```

## 创建通道

```shell
./network.sh createChannel    # 不指定通道名，通道名默认为mychannel
./network.sh createChannel -c channel1 # 指定通道名 channel1
```

## 安装链码

```shell
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

- `-ccn`指定链码名称 `chain code name`
- `-ccl` 指定编写链码的语言 `chain code language`
- `-ccp` 指定链码所在的目录 `chain code path`
- 其余的查看 `./network.sh help`

这一步在linux下，比如ubuntu,我当时设置了goproxy是在`~/.zshrc` 和`/etc/profile`中设置的，没有在`/etc/bash.bashrc`中设置(这里设置的环境变量对所有用户有效)，因为是使用`sudo`来跑命令,所以在`~/.zshrc` 和`/etc/profile`中设置的在终端里面`go env`查看是正常的，但安装链码的命令执行时还是下载依赖超时，因为根本没有使用`goproxy`。

还有加`sudo`后找不到`go`的坑,需要在`~/.zshrc`中添加

```PLAINTEXT
alias sudo='sudo env PATH=$PATH LD_LIBRARY_PATH=$LD_LIBRARY_PATH'
```

## 与fabric网络交互

**确保当前是在`test-network`目录**

```shell
export PATH=${PWD}/../bin:$PATH
```

添加这个环境变量是为了下面调用`peer`命令系统能找到`peer`指令，除了`peer`指令，还有其它一些指令，如下：

```shell
-rwxr-xr-x@ 1 mao  staff    15M Oct 26 23:58 configtxgen
-rwxr-xr-x@ 1 mao  staff    14M Oct 26 23:58 configtxlator
-rwxr-xr-x@ 1 mao  staff    11M Oct 26 23:58 cryptogen
-rwxr-xr-x@ 1 mao  staff    16M Oct 26 23:58 discover
-rwxr-xr-x@ 1 mao  staff    26M Jul  8 17:32 fabric-ca-client
-rwxr-xr-x@ 1 mao  staff    32M Jul  8 17:33 fabric-ca-server
-rwxr-xr-x@ 1 mao  staff    16M Oct 26 23:58 ledgerutil
-rwxr-xr-x@ 1 mao  staff    26M Oct 26 23:58 orderer
-rwxr-xr-x@ 1 mao  staff    12M Oct 26 23:58 osnadmin
-rwxr-xr-x@ 1 mao  staff    33M Oct 26 23:58 peer
```

还需要将`fabric-samples`代码库中的`FABRIC_CFG_PATH`设置为指向其中的`core.yaml`文件：

```shell
export FABRIC_CFG_PATH=$PWD/../config/
```

现在，可以设置环境变量，以允许作为Org1操作`peer` CLI：

```shell
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

`CORE_PEER_TLS_ROOTCERT_FILE`和`CORE_PEER_MSPCONFIGPATH`环境变量指向Org1的`organizations`文件夹中的的加密材料。 如果您使用 `./network.sh deployCC -ccl go` 安装和启动 asset-transfer (basic) 链码，您可以调用链码（Go）的 `InitLedger` 方法来赋予一些账本上的初始资产（如果使用 typescript 或者 javascript，例如 `./network.sh deployCC -l javascript`，你会调用相关链码的 `initLedger` 功能）。 运行以下命令用一些资产来初始化账本：

### 调用链码函数`InitLedger`

```shell
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
```

**Mac M1**中试验有很多坑，有说go版本要1.13，有说是Mac中的Docker Desktop的锅，准备再搞个**Linux环境**来玩了

### 查询

```shell
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

