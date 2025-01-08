---
title: Fabric-链码的基本结构和示例说明
tags:
  - fabric
  - chaincode
  - shim
  - 帐本
cover: 'https://s2.loli.net/2022/11/02/ePMCo3xnyr49zV8.png'
categories: 
  - 区块链
  - Fabric
abbrlink: 39166
date: 2022-11-02 10:32:33
---

# 链码结构

## 链码接口

链码启动必须通过调用 shim 包中的 Start 函数，传递一个类型为 Chaincode 的参数，该参数是一个接口类型，有两个重要的函数 Init 与 Invoke 。

```go
type Chaincode interface {
    Init(stub ChaincodeStubInterface) peer.Response
    Invoke(stub ChaincodeStubInterface) peer.Response
}
```

- Init: 在链码实例化或升级时被调用，完成初始化数据的工作
- Invoke: 更新或查询帐本数据状态时被调用，需要在此方法中**实现响应调用或查询的业务逻辑**

实际开发中，开发人员可以自行定义一个结构体，重写Chaincode接口的两个方法，并将两个方法指定为自定义的结构体的成员方法。相当于java语言里面写一个类实现链码的接口，实现接口中的`Init`和`Invoke`方法。

## 链码结构

```go
package main

// 引入必要的包
import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    pb "github.com/hyperledger/fabric/protos/peer"
)

// 声明一个结构体
type SimpleChaincode struct {
}

// 为结构体添加Init方法
func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
    // 在该方法中实现链码初始化或升级时的处理逻辑
    // 编写时可灵活使用stub中的API
}

// 为结构体添加Invoke方法
func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
    // 在该方法中实现链码运行中被调用或查询时的处理逻辑
    // 编写时可灵活使用stub中的API
}

// 主函数，需要调用shim.Start（）方法
func main() {
    err := shim.Start(new(SimpleChaincode))
    if err != nil {
        fmt.Printf("Error starting Simple chaincode: %s", err)
    }
}
```

- **shim**`: 用来访问/操作数据状态、事务上下文和调用`其它链`代码的API，链码通过`shim.ChaincodeStub`提供的方法来读取和修改帐本的状态。

- **peer**: 提供了链码执行后的响；应信息的API， `peer.Response`封装了响应信息

# 链码相关API

`shim`包提供了如下几种类型的接口：

- **参数解析AP**`：调用链码时需要给被调用的目标函数/方法传递参数，该 API 提供解析这些参数的方法，链码被调用时触发链码中的`Invoke`方法被调用，在`Invoke`内部，可以通过参数`stub`来获取目标函数/方法参数。
- **帐本状态数据操作API**: 该 API 提供了对账本数据状态进行操作的方法，包括对状态数据的查询及事务处理等
- **交易信息获取 API：**获取提交的交易信息的相关 API
- **对 PrivateData 操作的 API：** Hyperledger Fabric 在 1.2.0 版本中新增的对私有数据操作的相关 API
- **其他 API：**其他的 API，包括事件设置、调用其他链码操作

## 参数解析API

```go
// 返回调用链码时指定提供的参数列表（以字符串数组形式返回）
GetStringArgs() []string

// 返回调用链码时在交易提案中指定提供的被调用的函数名称及函数的参数列表（以字符串数组形式返回）
GetFunctionAndParameters() (function string, params []string)

// 返回提交交易提案时提供的参数列表（以字节串数组形式返回）
GetArgsSlice() ([]byte, error)

// 返回调用链码时在交易提案中指定提供的被调用的函数名称及函数的参数列表（以字节串数组形式返回）
GetArgs() [][]byte
```

一般使用 GetFunctionAndParameters() 及 GetStringArgs() 。

## 帐本数据状态操作API

```go
// 查询账本，返回指定键对应的值
GetState(key string) ([]byte, error)

// 尝试添加/更新账本中的一对键值
// 这一对键值会被添加到写集合中，等待 Committer 进一步确认，验证通过后才会真正写入到账本
PutState(key string, value []byte) error

// 尝试删除账本中的一对键值
// 同样，对该对键值删除会添加到写集合中，等待 Committer 进一步确认，验证通过后才会真正写入到账本
DelState(key string) error

// 查询指定范围的键值，startKey 和 endkey 分别指定开始（包括）和终止（不包括），当为空时默认是最大范围
// 返回结果是一个迭代器结构，可以按照字典序迭代每个键值对，最后需要调用 Close() 方法关闭
GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)

// 返回指定键的所有历史值。该方法的使用需要节点配置中打开历史数据库特性（ledger.history.enableHistoryDatabase=true）
GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error)

// 给定一组属性（attributes），将这些属性组合起来构造返回一个复合键
// 例如：CreateComositeKey("name-age",[]string{"Alice", "12"});
CreateCompositeKey(objectType string, attributes []string) (string, error)

// 将指定的复合键进行分割，拆分成构造复合键时所用的属性
SplitCompositeKey(compositeKey string) (string, []string, error)

// 根据局部的复合键（前缀）返回所有匹配的键值，即与账本中的键进行前缀匹配
// 返回结果是一个迭代器结构，可以按照字典序迭代每个键值对，最后需要调用 Close() 方法关闭
GetStateByPartialCompositeKey(objectType string, keys []string) (StateQueryIteratorInterface, error)

// 对(支持富查询功能的)状态数据库进行富查询，返回结果是一个迭代器结构，目前只支持 CouchDB
// 注意该方法不会被 Committer 重新执行进行验证，所以不能用于更新账本状态的交易中
GetQueryResult(query string) (StateQueryIteratorInterface, error)
```

**注意：** 通过 put 写入的数据状态不能立刻 get 到，因为 put 只是链码执行的模拟交易（防止重复提交攻击），并不会真正将状态保存到账本中，必须经过 Orderer 达成共识之后，将数据状态保存在区块中，然后保存在各 peer 节点的账本中。

## 交易信息相关API

```go
// 返回交易提案中指定的交易 ID。
// 一般情况下，交易 ID 是客户端提交提案时由 Nonce 随机串和签名者身份信息哈希产生的数字摘要
GetTxID() string

// 返回交易提案中指定的 Channel ID
GetChannelID() string

// 返回交易被创建时的客户端打上的的时间戳
// 这个时间戳是直接从交易 ChannnelHeader 中提取的，所以在所以背书节点处看到的值都相同
GetTxTimestamp() (*timestamp.Timestamp, error)

// 返回交易的 binding 信息
// 交易的 binding 信息是将交提案的 nonse、Creator、epoch 等信息组合起来哈希得到数字摘要
GetBinding() ([]byte, error)

// 返回该 stub 的 SignedProposal 结构，包括了跟交易提案相关的所有数据
GetSignedProposal() (*pb.SignedProposal, error)

// 返回该交易提交者的身份信息（用户证书）
// 从 SignedProposal 中的 SignatureHeader.Creator 提取
GetCreator() ([]byte, error)

// 返回交易中带有的一些临时信息
// 从 ChaincodeProposalPayload.transient 提取，可以存放与应用相关的保密信息，该信息不会被写入到账本
GetTransient() (map[string][]byte, error)
```

## 对 PrivateData 操作的 API

```go
// 根据指定的 key，从指定的私有数据集中查询对应的私有数据
GetPrivateData(collection, key string) ([]byte, error)

// 将指定的 key 与 value 保存到私有数据集中
PutPrivateData(collection string, key string, value []byte) error

// 根据指定的 key 从私有数据集中删除相应的数据
DelPrivateData(collection, key string) error

// 根据指定的开始与结束 key 查询范围（不包含结束key）内的私有数据
GetPrivateDataByRange(collection, startKey, endKey string) (StateQueryIteratorInterface, error)

// 根据给定的部分组合键的集合，查询给定的私有状态
GetPrivateDataByPartialCompositeKey(collection, objectType string, keys []string) (StateQueryIteratorInterface, error)

// 根据指定的查询字符串执行富查询 （只支持支持富查询的 CouchDB）
GetPrivateDataQueryResult(collection, query string) (StateQueryIteratorInterface, error)
```

## 其他 API

```go
// 设定当这个交易在 Committer 处被认证通过，写入到区块时发送的事件（event），一般由 Client 监听
SetEvent(name string, payload []byte) error

// 调用另外一个链码的 Invoke 方法
// 如果被调用链码在同一个通道内，则添加其读写集合信息到调用交易；否则执行调用但不影响读写集合信息
// 如果 channel 为空，则默认为当前通道。目前仅限读操作，同时不会生成新的交易
InvokeChaincode(chaincodeName string, args [][]byte, channel string) pb.Response
```

# 链码开发

## 帐户转帐

```go
package main

import (
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric-chaincode-go/shim"
	"github.com/hyperledger/fabric-protos-go/peer"
)

// 声明一个结构体
type SimpleChaincode struct {
}

// 初始化数据状态， 实例化/升级链码时被自动调用
func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) peer.Response {
	// println 函数的输出信息会出现在链码容器的日志中
	fmt.Println("ex02 Init")

	// 获取用户传递给调用链码的所需参数
	_, args := stub.GetFunctionAndParameters()

	var A, B string    // 两个帐户
	var Aval, Bval int // 两个帐户的余额
	var err error

	// 检查合法性，检查参数数量是否为4个，如果不是，则返回错误信息
	if len(args) != 4 {
		return shim.Error("Incorrect number of arguments. Expecting 4")
	}

	A = args[0]                       // 帐户A用户名
	Aval, err = strconv.Atoi(args[1]) // 帐户A余额
	if err != nil {
		return shim.Error("Expecting integer value for asset holding")
	}

	B = args[2]                       // 账户B用户名
	Bval, err = strconv.Atoi(args[3]) // 帐户B余额
	if err != nil {
		return shim.Error("Expecting integer value for asset holding")
	}

	fmt.Printf("Aval = %d, Bval = %d\n", Aval, Bval)

	// 将账户A的状态写入账本中
	err = stub.PutState(A, []byte(strconv.Itoa(Aval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	// 将帐户B的状态写入账本
	err = stub.PutState(B, []byte(strconv.Itoa(Bval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	// 一切成功，返回
	return shim.Success(nil)
}

// 对帐户数据进行操作时（query、invoke）被自动调用
func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
	fmt.Println("ex02 Invoke")
	// 获取用户传递给调用链码的函数名称及参数
	function, args := stub.GetFunctionAndParameters()

	// 对获取到的函数名称进行判断
	if function == "invoke" {
		// 调用invoke函数实现转帐操作
		return t.invoke(stub, args)
	} else if function == "delete" {
		// 调用delete函数实现帐户注销
		return t.delete(stub, args)
	} else if function == "query" {
		// 调用query实现帐户查询操作
		return t.query(stub, args)
	}
	// 调用的链码函数名出错，返回shim.Error()
	return shim.Error("Invalid invoke function name. Expecting \"invoke\" \"delete\" \"query\"")
}

// 帐户间转钱
func (t *SimpleChaincode) invoke(stub shim.ChaincodeStubInterface, args []string) peer.Response {
	var A, B string    // 帐户A和B
	var Aval, Bval int // 帐户余额
	var X int          // 转账金额
	var err error

	if len(args) != 3 {
		return shim.Error("Incorrect number of arguments. Expecting 3")
	}

	A = args[0] // 帐户A用户名
	B = args[1] // 帐户B用户名

	// 从账本中获取A的余额
	Avalbytes, err := stub.GetState(A)
	if err != nil {
		return shim.Error("Failed to get state")
	}
	if Avalbytes == nil {
		return shim.Error("Entity not found")
	}
	Aval, _ = strconv.Atoi(string(Avalbytes))

	// 从帐本中获取B的余额
	Bvalbytes, err := stub.GetState(B)
	if err != nil {
		return shim.Error("Failed to get state")
	}
	if Bvalbytes == nil {
		return shim.Error("Entity not found")
	}
	Bval, _ = strconv.Atoi(string(Bvalbytes))

	// X 为转账金额
	X, err = strconv.Atoi(args[2])
	if err != nil {
		return shim.Error("Invalid transaction amount, expecting a integer value")
	}

	// 转帐
	Aval = Aval - X
	Bval = Bval + X
	fmt.Printf("Aval = %d, Bval = %d\n", Aval, Bval)

	// 更新转帐后账本中A余额
	err = stub.PutState(A, []byte(strconv.Itoa(Aval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	// 更新转帐后账本中B余额
	err = stub.PutState(B, []byte(strconv.Itoa(Bval)))
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

// 帐户注销
func (t *SimpleChaincode) delete(stub shim.ChaincodeStubInterface, args []string) peer.Response {
	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting 1")
	}

	A := args[0] // 帐户用户名

	// 从账本中删除该帐户状态
	err := stub.DelState(A)
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

// 帐户查询
func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) peer.Response {
	var A string
	var err error

	if len(args) != 1 {
		return shim.Error("Incorrect number of arguments. Expecting name of the person to query")
	}

	A = args[0] // 帐户用户名

	// 从帐户中获取该帐户余额
	Avalbytes, err := stub.GetState(A)
	if err != nil {
		jsonResp := "{\"Error\": \"Failed to get state for " + A + "\"}"
		return shim.Error(jsonResp)
	}
	if Avalbytes == nil {
		jsonResp := "{\"Error\":\"Nil amount for " + A + "\"}"
		return shim.Error(jsonResp)
	}

	jsonResp := "{\"Name\":\"" + A + "\",\"Amount\":\"" + string(Avalbytes) + "\"}"
	fmt.Printf("Query Response:%s\n", jsonResp)
	// 返回转帐金额
	return shim.Success(Avalbytes)
}

func main() {
	err := shim.Start(new(SimpleChaincode))
	if err != nil {
		fmt.Printf("Error starting Simple chaincode: %s", err)
	}
}
```

