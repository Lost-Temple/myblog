---
title: Fabric-wasmcc介绍
tags:
  - wasm
  - chaincode
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 17370
date: 2022-11-03 10:52:12
---

# wasmcc

全称：`Hyperledger Fabric WASM Chaincode`，是由`hypterledger-labs`提供的一个链码。作用是可以把使用rust/C编写代码生成`wasm`字节码，把wasm字节码部署到`fabric`中，然后使用wasmcc作为代理调用wasm中的函数。fabric以此方式来支持`wasm chaincode`

## 步骤
- 1. `wasmcc`部署到`fabric`中，扮演`wasm`代码调用的统一入口，相当于wasm代码的调用代理。
`wasmcc`的部署和其它的普通链码的部署方式一样，`wasmcc`没有实例化参数，通过peer cli部署的命令如下：
```shell
peer chaincode install -n wasmcc -v 1.0 -p github.com/chaincode/wasmer-chaincode-test/wasmcc
```
```shell
peer chaincode instantiate -n wasmcc -v 1.0 -C <channel-name> -c '{"Args":[]}' -o <orderer-address> --tls --cafile <orderer-ca>
```
- 2. 通过调用`wasmcc`的 `create` 方法创建`wasm chaincode`
- 3. **通过调用`wasmcc`的`execute`方法调用`wasm chaincode`**
- 4. 通过调用`wasmcc`的`installedChaincodes`方法获取目前fabric上部署的所有`wasm chiancode`

# wasmcc 代码解读

## 基础结构(fabric 链码的基本结构)


{% mermaid %}
classDiagram
class WASMChaincode {
	+Init(stub shim.ChaincodeStubInterface) Response
	+Invoke(stub shim.ChaincodeStubInterface) Response
	-create(stub shim.ChaincodeStubInterface, args []string) Response
	-execute(stub shim.ChaincodeStubInterface, args []string) Response
	-installedChaincodes(stub shim.ChaincodeStubInterface, args []string) Response
}
class VirtualMachine {
...
	+ImportResolver ImportResolver

}
class ImportResolver {
	<<interface>>
	+ResolveFunc(module, field string) FunctionImport
	+ResolveGlobal(module, field string) int64
}
class Resolver {
	-string chaincodeName
	-shim.ChaincodeStubInterface stub          
	-[]string args          
	-[]byte result        
	-[]byte errMsg        
	+ResolveFunc(module, field string) FunctionImport
	+ResolveGlobal(module, field string) int64
}
class ChaincodeStubInterface {
	<<interface>>
	...
}
class ChaincodeStub {
	...
}

ChaincodeStubInterface <|.. ChaincodeStub
ImportResolver <|.. Resolver
WASMChaincode <-- VirtualMachine
WASMChaincode ..> ChaincodeStubInterface
VirtualMachine ..> ImportResolver
Resolver --> WASMChaincode

{% endmermaid %}

**wasm链码的创建流程（create/execute/installedChaincodes三个流程类似）**

{% mermaid %}
sequenceDiagram
	participant gRpc Client
	participant WASMChaincode
	participant ChaincodeStub
  gRpc Client ->> WASMChaincode: 调用链码的create函数（触发链码Invoke）
  WASMChaincode ->> ChaincodeStub: 获取函数及函数参数数组
  ChaincodeStub -->> WASMChaincode: 返回被调用函数及函数参数数组
  WASMChaincode ->> WASMChaincode: 调用自身create函数
  WASMChaincode ->> ChaincodeStub: 检查链码是否存在
  ChaincodeStub -->> WASMChaincode: 链码已存在
  WASMChaincode -->> gRpc Client: 失败，链码已存在
  ChaincodeStub -->> WASMChaincode: 链码不存在，继续执行
  WASMChaincode ->> WASMChaincode: 对wasm字节码进行解码，得能解压后的字节流
  WASMChaincode ->> WASMChaincode: 给Resolver对象赋值（里面保存了链码名，stub, 函数参数数组）
  WASMChaincode ->> WASMChaincode: 调用runWASM，传入wasm字节流,函数名"init",参数个数，以及Resolver对象
  WASMChaincode -->> gRpc Client: 调用runWASM失败立即返回
  WASMChaincode ->> WASMChaincode: 创建"chaincodeData"和链码名的组合键
  WASMChaincode ->> ChaincodeStub: "chaincodeData"和链码名的组合为键，wasm字节流为值，存入状态库
  ChaincodeStub -->> WASMChaincode: 返回
  WASMChaincode -->> gRpc Client: 返回

{% endmermaid %}

### 定义一个结构体

```go
type WASMChaincode struct {
}
```

## Init方法

```go
func (t *WASMChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	logger.Infof("Init invoked")
	return shim.Success(nil)
}
```

链码创建或升级时被调用，`wasmcc `的`Init`内部只记录了一下日志，没有其它多余的操作

## Invoke方法

```go
func (t *WASMChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	logger.Infof("Invoke function")
	function, args := stub.GetFunctionAndParameters()
	logger.Debugf("Invoke function %s with args %v", function, args)

	if function == "create" {
		// Create a new wasm chaincode
		return t.create(stub, args)
	} else if function == "execute" {
		// execute wasm chaincode
		return t.execute(stub, args)
	} else if function == "installedChaincodes" {
		// get the list of installed wasm chaincode
		return t.installedChaincodes(stub, args)
	}

	return shim.Error("Invalid invoke function name. Expecting \"execute\" \"create\" \"query\"")
}
```

这个方法的功能：

- 1. 创建wasm chaincode: 调用函数`create`

     ```go
     // Store a new wasm chaincode in state. Receives chaincode name and wasm file encoded in hex
     func (t *WASMChaincode) create(stub shim.ChaincodeStubInterface, args []string) pb.Response {
     	logger.Infof("Create function")
     	if len(args) < 2 {
     		return shim.Error("Incorrect number of arguments. Expecting atleast 2 arguments")
     	}
     
     	chaincodeName := args[0]
     	logger.Infof("Installing wasm chaincode: " + chaincodeName)
     
     	//check if same chaincode name is already present
     	chaincodeFromState, err := stub.GetState(chaincodeName)
     	if chaincodeFromState != nil {
     		return shim.Error(ChaincodeExists)
     	}
     
     	//Decode the chaincode
     	chaincodeHexEncoded := args[1]
     	chaincodeDecoded, err := decodeReceivedWASMChaincode(chaincodeHexEncoded)
     
     	if err != nil {
     		return shim.Error(err.Error())
     	}
     
     	//Initialize global variables for exported wasm functions
     	r := Resolver{
     		chaincodeName, stub, args[2:], nil, nil,
     	}
     
     	result := runWASM(chaincodeDecoded, "init", len(args)-2, &r)
     
     	logger.Infof("Init Response:%d\n", result)
     
     	if result != 0 {
     		return shim.Error("Chaincode init invocation failed")
     	}
     
     	// Store the chaincode in
     	ledgerChaincodeKey, err := stub.CreateCompositeKey(chaincodeStoreIndex, []string{chaincodeName})
     	err = stub.PutState(ledgerChaincodeKey, chaincodeDecoded)
     	if err != nil {
     		s := fmt.Sprintf(UnknownError, err.Error())
     		return shim.Error(s)
     	}
     	return shim.Success([]byte("Success! Installed wasm chaincode"))
     }
     ```

- 2. 执行wasm chaincode 中的函数

     ```go
     func (t *WASMChaincode) execute(stub shim.ChaincodeStubInterface, args []string) pb.Response {
     
     	if len(args) < 2 {
     		return shim.Error("Incorrect number of arguments. Expecting chaincode name to invoke")
     	}
     
     	chaincodeName := args[0]
     	funcToInvoke := args[1]
     
     	//Initialize global variables for exported wasm functions
     	r := Resolver{
     		chaincodeName, stub, args[2:], nil, nil,
     	}
     
     	// Get the state from the ledger
     	ledgerChaincodeKey, _ := stub.CreateCompositeKey(chaincodeStoreIndex, []string{chaincodeName})
     	Chaincodebytes, _ := stub.GetState(ledgerChaincodeKey)
     	if Chaincodebytes == nil {
     		jsonResp := "{\"Error\":\"No Chaincode for " + chaincodeName + "\"}"
     		return shim.Error(jsonResp)
     	}
     
     	result := runWASM(Chaincodebytes, funcToInvoke, len(args)-2, &r)
     
     	logger.Infof("Invoke Response:%d\n", result)
     	return txnResult(result, r.result)
     }
     ```


- 3. 获取已安装的`wasm chaincode`

     ```go
     func (t *WASMChaincode) installedChaincodes(stub shim.ChaincodeStubInterface, args []string) pb.Response {
     
     	// Get all chaincodes from the ledger
     	installedChaincodeResultsIterator, err := stub.GetStateByPartialCompositeKey(chaincodeStoreIndex, []string{})
     	if err != nil {
     		return shim.Error(err.Error())
     	}
     
     	// Iterate through result set and get chaincode names
     	var i int
     	var installedChaincodeNamesList = ""
     
     	for i = 0; installedChaincodeResultsIterator.HasNext(); i++ {
     		// Note that we don't get the value (2nd return variable), we'll just get the marble name from the composite key
     		responseRange, err := installedChaincodeResultsIterator.Next()
     		if err != nil {
     			return shim.Error(err.Error())
     		}
     
     		// get the color and name from color~name composite key
     		objectType, compositeKeyParts, err := stub.SplitCompositeKey(responseRange.Key)
     		if err != nil {
     			return shim.Error(err.Error())
     		}
     		returnedChaincodeName := compositeKeyParts[0]
     		logger.Infof("- found a chaincode from index:%s name:%s\n", objectType, returnedChaincodeName)
     		installedChaincodeNamesList += returnedChaincodeName
     		installedChaincodeNamesList += "\n"
     
     	}
     
     	logger.Infof("Invoke Response:%s\n", installedChaincodeNamesList)
     	return shim.Success([]byte(installedChaincodeNamesList))
     }
     ```
