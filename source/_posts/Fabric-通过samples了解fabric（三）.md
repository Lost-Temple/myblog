---
title: Fabric-通过samples了解fabric（三）
tags:
  - wasmcc
  - wasm chaincode
  - file-encoder
  - wasm pusher
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 12153
date: 2022-11-07 14:44:44
---

**注：以下命令在`test-network`目录下执行**
```shell
export PATH=${PWD}/../bin:$PATH         # peer 命令所在目录
export FABRIC_CFG_PATH=$PWD/../config/  # `core.yaml` 配置文件所在目录
```

# 部署`wasmcc`
```shell
./network.sh deployCC -ccn wasmcc -ccp /Users/mao/work/wasm/opensource/hyperledger/fabric-chaincode-wasm/wasmcc -ccv 1.0 -ccl go
```


# 调用`wasmcc`部署`wasm chaincode`

首先准备好一个wasm文件，使用工具把wasm文件的二进制数据转成十六进制表示（字符串），
然后通过以下命令部置这个`wasm chaincode`,**注：命令中的`wasm二进制数据的十六进制表示字符串`需要按实际的数据进行替换。这个把wasm字节码转为十六进制表示（字符串）的工具是[file-encoder](https://github.com/hyperledger-labs/fabric-chaincode-wasm/tree/main/tools/file-encoder)

```shell
peer chaincode invoke -o localhost:7050 --tls true --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n wasmcc --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["create","balancewasm","wasm二进制数据的十六进制表示字符串","account1","100","account2","1000"]}'
```

# 调用命令查看目前已部署的`wasm chaincode`
```shell
peer chaincode query -C mychannel -n wasmcc -c '{"Args":["installedChaincodes"]}'
```

# 通过`wasmcc`查询`wasm chaincode`中的`account1`的余额
```shell
peer chaincode query -C mychannel -n wasmcc -c '{"Args":["execute","balancewasm","query","account1"]}'
```


# 通过`wasmcc`调用`wasm chaincode`中的函数`invoke`
```shell
peer chaincode invoke -o localhost:7050 --tls true --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n wasmcc --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["execute","balancewasm","invoke","account2","account1","10"]}'
``` 

# 通过工具`WASM Pusher`安装`wasm chaincode`
`wasmcc`支持3种格式文件的安装，`.wasm`格式、`.zip`格式、`wasm`字节`十六进制表示`的字符串内容。
使用命令如下：
```shell
./wasm-pusher -n balancewasm -w ../../sample-wasm-chaincode/chaincode_example02/rust/app_main.wasm -u User1 -a a,100,b,100
```
`WASM Pusher`[详见](https://github.com/hyperledger-labs/fabric-chaincode-wasm/tree/main/tools/wasm-pusher)

