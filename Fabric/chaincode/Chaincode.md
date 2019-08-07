[TOC]

[链码的概念与使用](<https://blog.csdn.net/qq_37133717/article/details/80932904> )



## 一、写链码

每个chaincode程序都必须实现 **chiancode接口** ，接口中的方法会在响应传来的交易时被调用。特别地，`Init`（初始化）方法会在chaincode接收到`instantiate`（实例化）或者`upgrade`(升级)交易时被调用，进而使得chaincode顺利执行必要的初始化操作，包括初始化应用的状态；`Invoke`（调用）方法会在响应`invoke`（调用）交易时被调用以执行交易。 



对于每一个chaincode，它都会实现预定义的**chaincode接口**，特别是`Init`和`Invoke`函数接口。所以我们首先为我们的chaincode引入必要的依赖。我们将在此引入chaincode shim package和**peer protobuf package**。 

```go
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)
```



(1) 初始化Chaincode

链码必须有的2个方法

```go
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    // do something
    return shim.Success(nil)
}

func (t TokenChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	function, args := stub.GetFunctionAndParameters()

	if function == "" {
		return shim.Error("no function err")
	} else if function == "balance" {
		return t.Balance(stub, args)
	} else if function == "allowance" {
		return t.Allowance(stub, args)
	} 

	log.Errorf("%s was not found", function)
	return shim.Error(fmt.Sprintf("%s was not found", function))
}
```

值得留意的是chaincode升级同样会调用该函数。当我们编写的chaincode会升级现有chaincode时，需要确保适当修正Init函数。特别地，如果没有“迁移”操作或其他需要在升级中初始化的东西，那么就提供一个空的“Init”方法。

## 二、打包链码

**(1) 什么是智能合约包：**

- 智能合约由智能合约部署规范（ChaincodeDeploymentSpec）或CDS定义。CDS是根据代币和其他属性（如名称和版本）定义的智能合约包
- 一个可选的实例化策略。
- 由“拥有”智能合约的实体的一组签名

**(2) 签名的目的**

- 为了建立智能和的所有权
- 允许对包的内容进行验证
- 允许检测包是否篡改

**(3) 怎样创建package(包)**

打包合约有两种方法。

- 一种是当想要一个智能合约拥有多个所有者时，需要多个身份标志为该合约签名。首先创建一个已签名的智能合约（一个签署的CDS）,让后通过序列的方式将其传递给其他所有者来签署。
- 另一种，正在发行安装事务的节点的身份签名时部署己签署的 CDS

下面是打包一个智能合约的包

```
peer chaincode package -n $CC_NAME -p github.com/chaincode/justincc/demo1 -v $CC_VERSION -s -S -i "AND('Org1MSP.admin')" ccpack.out
```

参数说明：

- `-s`  参数是指可以创建由多个所有者签署的包，而不是简单的创建一个未处理成修饰过的CDS。当指定了-s时，如果其他所有者需要签名，也必须指定-s参数。否则，这个过程会创建一个除了CDS实例化策略之外的已签署的CDS
- `-S` 参数使用在core.yaml中有LocakMspid属性值标志的MSP来指示该程序的签名，这个参数是可选的，如果一个包没有签名的情况下创建的，那么它就不能由任何其他所有者使用signpackage命令来签署
- `-i`参数可选的，即指定智能合约实例化策略。实例化策略和背书策略具有相同的格式。实例化链码一节中有介绍。如果没有提供策略，则使用默认策略，这将默认允许channel的任何MSP的admin身份来实例化智能合约

**(4) 签名package(包)**

一个已经被签名的智能合约包在创建时候可以交友其他所有者检查并签名，这个工作流程支持智能合约包带外签名。

```
peer chaincode signpackage ccpack.out signedccpack.out
```

> `ccpack.out` 和`signedccpack.out`分别是输入包和输出包, 输出包包含了使用本地MSP签名的包的附加签名

输出包又叫做SignedCDS,包含了

- CDS包含了智能合约的源码，名称和版本信息
- 智能合约的实例化策略，即背书策略。
- 智能合约的所有者列表，以背书的方式定义

## 三、部署链码

部署链码，是在cli容器中进行的。所有需要执行下面的命令，进入cli容器内部：

```
sudo docker exec -it cli bash
```

### 3.0 环境变量

设置临时环境变量，方便下面的命令，反复执行的时候，修改的麻烦。

```
export CHANNEL_NAME="mychannel"
export CC_NAME="demo1"
export CC_VERSION="1.0"
export ORDERER_CA="/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
```

- CHANNEL_NAME 通道名
- CC_NAME  智能合约的名称
- CC_VERSION  合约的版本
- ORDERER_CA  Orderer服务的证书

### 3.1 安装链码

安装交易将智能合约的源代码打包成一种指定的格式，称为`ChaincodeDeploymentSpec`(智能合约部署规范或CDS)，并将其安装到运行该智能合约的Peer节点上。

```
peer chaincode install -n $CC_NAME -v $CC_VERSION -p github.com/chaincode/justincc/demo1
```

- `-p`参数是链码的源代码路径，根据`$GOPATH/src`来寻址的

安装交易实质上：是将一个签署的提案发送到生命周期系统智能合约（Lifecycle System chaincode, LSCC）。

注意：必须在channel中的每个背书节点上安装智能合约，才能正常运行智能合约。

### 3.2 实例化链码

```
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -c '{"Args":["init","0x4e9ce36e442e55ecd9025b9a6e0d88485d628a67", "10000"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"
```

注意 `-P`选项，设置背书策略，

```
EXPR(E[, E ... ])
```

EXPR 使用 AND OR其中之作为表达式， 要么是一个，要么一个 EXPR 的嵌套结构

- ( l ) `AND(’Org1.member，’Org2.member’，’Org3.member’）`三个主体必须同时背书并认可
  签名
- (2) `OR(’Org1.member,'Org1.member'）`两个主体中的任意一个背书并认可签名
- (3) `OR(’Orgl.member', AND('Org2.member','Org3.member’))` 主体1背书 认可签名或者主2
  和主体3同时背书并认可签名

### 3.3 查询

智能合约可以直接访问本身的状态：

```
 peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["balanceOf","0x4e9ce36e442e55ecd9025b9a6e0d88485d628a67"]}'
```

正常情况下，不同智能合约是不能直接访问的。当它们在同一个网络中，可以通过给与适当的权限，达到跨合约访问的目的，不同通道直接，只能查询状态。

### 3.4 执行

```
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt -c '{"Args":["transfer","0x4e9ce36e442e55ecd9025b9a6e0d88485d628a67","0x742d35cc6634c0532925a3b844bc454e4438f44e","1000"]}'
```

执行后，再执行查询，结果应该是9000,

下面是一个需要2个节点背书的链码demo，全操作：

```
export CHANNEL_NAME="mychannel"
export CC_NAME="demo1"
export CC_VERSION="1.0"
export CC_PATH="github.com/chaincode/justincc/demo1"
export ORDERER_CA="/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"


# 在节点 peer0.org1 安装链码
peer chaincode install -n $CC_NAME -v $CC_VERSION -p $CC_PATH

# 实例化链码
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -c '{"Args":["init","0x4e9ce36e442e55ecd9025b9a6e0d88485d628a67", "10000"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"

# 查询已经实例化的链码
peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["balanceOf","0x4e9ce36e442e55ecd9025b9a6e0d88485d628a67"]}'

# 切换到 peer0.org2 节点
CORE_PEER_LOCALMSPID="Org2MSP"
CORE_PEER_ADDRESS=peer0.org2.example.com:7051
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt

# 在节点 peer0.org2 安装链码
peer chaincode install -n $CC_NAME -v $CC_VERSION -p $CC_PATH

# 调用，同时向2个组织的节点请求背书，发起函数调用。修改账本
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile $ORDERER_CA -C $CHANNEL_NAME -n $CC_NAME --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["transfer","0x4e9ce36e442e55ecd9025b9a6e0d88485d628a67","0x742d35cc6634c0532925a3b844bc454e4438f44e","1000"]}'

# 查询已经实例化的链码
peer chaincode query -C $CHANNEL_NAME -n $CC_NAME -c '{"Args":["balanceOf","0x742d35cc6634c0532925a3b844bc454e4438f44e"]}'
```

### 3.5 升级链码

在升级之前，必须将智能合约的新版本安装在需要的背书Peer上。升级类似实例化的交易，它将智能合约新版本绑定到Channel。不会影响其合约。旧版本的智能合约并不会自动删除。请自行管理旧版本的合约。

升级合约，是根据当前的智能合约实例化中指定的策略进行检查的，而不是新版的策略。意味着升级合约的这笔交易还是按照旧合约的背书策略进行的。

升级活成中，调用chaincode Init 函数执行任何和数据相关的更新或其他操作，因此在升级智能合约时必须注意避免重新设置状态。

### 3.6 调试

```
export FABRIC_CFG_PATH="/home/justin/go/src/github.com/hyperledger/fabric/sampleconfig"
```

启动调试

```
peer node start --peer-chaincodedev
```



## 四、查看记录

### 4.1 容器日志

检查每个独立的链码服务容器来查看每个容器内的分隔的交易。下面是每个链码服务容器的日志的组合：

```
sudo docker logs dev-peer0.org1.example.com-demo1-1.0
```

获取的是链码中自定义的print语句输出的结果。

### 4.2 链信息

```
peer channel getinfo -c mychannel
```

打印结果

```
Blockchain info: {"height":6,"currentBlockHash":"3WXFNm1NazzMlT4eSUfZGFrxuZKb8/TK/VtDg6yhz8U=","previousBlockHash":"KP20sPqasvkejG1PcJk9qzcL2vDAXIeS2BtJaoeOPv0="}
```

### 4.3 块数据

分别查询对应的块数据，以文件的形式保存在当前工作目录

```
peer channel fetch newest -c mychannel
peer channel fetch oldest -c mychannel
peer channel fetch config -c mychannel
peer channel fetch number -c mychannel
```

### 4.4 交易信息

## 五、附录

- 不能使用不确定性的变聋作为计算的输入 比如不能采用随机数或者获取系统当前时间等 链码会在不同的节点上多次运行，但是运行的时间并不严格一致，环境的查询导致不同节点上执行的结果不一致，最终无法达成共识
- 避免调用外部数据接口导致重复计算：比如多个节点多次调用外部写数据的接口，可能会导致区块链外部重复计算 这种情况多个链码的执行结果可能是一致的，不会导致共识失败，但会影响外部的一致性，这是一种逻辑错误

## 九、附录

### 链码调用链码

```go
/*
Copyright IBM Corp. All Rights Reserved.

SPDX-License-Identifier: Apache-2.0
*/

package example04

import (
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
)

// This chaincode is a test for chaincode invoking another chaincode - invokes chaincode_example02

// SimpleChaincode example simple Chaincode implementation
type SimpleChaincode struct{}

func toChaincodeArgs(args ...string) [][]byte {
	bargs := make([][]byte, len(args))
	for i, arg := range args {
		bargs[i] = []byte(arg)
	}
	return bargs
}

// Init takes two arguments, a string and int. These are stored in the key/value pair in the state
func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	var event string // Indicates whether event has happened. Initially 0
	var eventVal int // State of event
	var err error
	_, args := stub.GetFunctionAndParameters()
	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	// Initialize the chaincode
	event = args[0]
	eventVal, err = strconv.Atoi(args[1])
	if err != nil {
		return shim.Error("Expecting integer value for event status")
	}
	fmt.Printf("eventVal = %d\n", eventVal)

	err = stub.PutState(event, []byte(strconv.Itoa(eventVal)))
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

// Invoke invokes another chaincode - chaincode_example02, upon receipt of an event and changes event state
func (t *SimpleChaincode) invoke(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var event string // Event entity
	var eventVal int // State of event
	var err error

	if len(args) != 3 && len(args) != 4 {
		return shim.Error("Incorrect number of arguments. Expecting 3 or 4")
	}

	chainCodeToCall := args[0]
	event = args[1]
	eventVal, err = strconv.Atoi(args[2])
	if err != nil {
		return shim.Error("Expected integer value for event state change")
	}
	channelID := ""
	if len(args) == 4 {
		channelID = args[3]
	}

	if eventVal != 1 {
		fmt.Printf("Unexpected event. Doing nothing\n")
		return shim.Success(nil)
	}

	f := "invoke"
	invokeArgs := toChaincodeArgs(f, "a", "b", "10")
	response := stub.InvokeChaincode(chainCodeToCall, invokeArgs, channelID)
	if response.Status != shim.OK {
		errStr := fmt.Sprintf("Failed to invoke chaincode. Got error: %s", string(response.Payload))
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}

	fmt.Printf("Invoke chaincode successful. Got response %s", string(response.Payload))

	// Write the event state back to the ledger
	err = stub.PutState(event, []byte(strconv.Itoa(eventVal)))
	if err != nil {
		return shim.Error(err.Error())
	}

	return response
}

func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var event string // Event entity
	var err error

	if len(args) < 1 {
		return shim.Error("Incorrect number of arguments. Expecting entity to query")
	}

	event = args[0]
	var jsonResp string

	// Get the state from the ledger
	eventValbytes, err := stub.GetState(event)
	if err != nil {
		jsonResp = "{\"Error\":\"Failed to get state for " + event + "\"}"
		return shim.Error(jsonResp)
	}

	if eventValbytes == nil {
		jsonResp = "{\"Error\":\"Nil value for " + event + "\"}"
		return shim.Error(jsonResp)
	}

	if len(args) > 3 {
		chainCodeToCall := args[1]
		queryKey := args[2]
		channel := args[3]
		f := "query"
		invokeArgs := toChaincodeArgs(f, queryKey)
		response := stub.InvokeChaincode(chainCodeToCall, invokeArgs, channel)
		if response.Status != shim.OK {
			errStr := fmt.Sprintf("Failed to invoke chaincode. Got error: %s", err.Error())
			fmt.Printf(errStr)
			return shim.Error(errStr)
		}
		jsonResp = string(response.Payload)
	} else {
		jsonResp = "{\"Name\":\"" + event + "\",\"Amount\":\"" + string(eventValbytes) + "\"}"
	}
	fmt.Printf("Query Response: %s\n", jsonResp)

	return shim.Success([]byte(jsonResp))
}

// Invoke is called by fabric to execute a transaction
func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	function, args := stub.GetFunctionAndParameters()
	if function == "invoke" {
		return t.invoke(stub, args)
	} else if function == "query" {
		return t.query(stub, args)
	}

	return shim.Error("Invalid invoke function name. Expecting \"invoke\" \"query\"")
}
```



### 链码查询链码

```go
/*
Copyright IBM Corp. All Rights Reserved.

SPDX-License-Identifier: Apache-2.0
*/

package example05

import (
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
)

// This chaincode is a test for chaincode querying another chaincode - invokes chaincode_example02 and computes the sum of a and b and stores it as state

// SimpleChaincode example simple Chaincode implementation
type SimpleChaincode struct{}

func toChaincodeArgs(args ...string) [][]byte {
	bargs := make([][]byte, len(args))
	for i, arg := range args {
		bargs[i] = []byte(arg)
	}
	return bargs
}

// Init takes two arguments, a string and int. The string will be a key with
// the int as a value.
func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
	var sum string // Sum of asset holdings across accounts. Initially 0
	var sumVal int // Sum of holdings
	var err error
	_, args := stub.GetFunctionAndParameters()
	if len(args) != 2 {
		return shim.Error("Incorrect number of arguments. Expecting 2")
	}

	// Initialize the chaincode
	sum = args[0]
	sumVal, err = strconv.Atoi(args[1])
	if err != nil {
		return shim.Error("Expecting integer value for sum")
	}
	fmt.Printf("sumVal = %d\n", sumVal)

	// Write the state to the ledger
	err = stub.PutState(sum, []byte(strconv.Itoa(sumVal)))
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(nil)
}

// Invoke queries another chaincode and updates its own state
func (t *SimpleChaincode) invoke(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var sum, channelName string // Sum entity
	var Aval, Bval, sumVal int  // value of sum entity - to be computed
	var err error

	if len(args) < 2 {
		return shim.Error("Incorrect number of arguments. Expecting atleast 2")
	}

	chaincodeName := args[0] // Expecting name of the chaincode you would like to call, this name would be given during chaincode install time
	sum = args[1]

	if len(args) > 2 {
		channelName = args[2]
	} else {
		channelName = ""
	}

	// Query chaincode_example02
	f := "query"
	queryArgs := toChaincodeArgs(f, "a")

	//   if chaincode being invoked is on the same channel,
	//   then channel defaults to the current channel and args[2] can be "".
	//   If the chaincode being called is on a different channel,
	//   then you must specify the channel name in args[2]

	response := stub.InvokeChaincode(chaincodeName, queryArgs, channelName)
	if response.Status != shim.OK {
		errStr := fmt.Sprintf("Failed to query chaincode. Got error: %s", response.Payload)
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}
	Aval, err = strconv.Atoi(string(response.Payload))
	if err != nil {
		errStr := fmt.Sprintf("Error retrieving state from ledger for queried chaincode: %s", err.Error())
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}

	queryArgs = toChaincodeArgs(f, "b")
	response = stub.InvokeChaincode(chaincodeName, queryArgs, channelName)
	if response.Status != shim.OK {
		errStr := fmt.Sprintf("Failed to query chaincode. Got error: %s", response.Payload)
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}
	Bval, err = strconv.Atoi(string(response.Payload))
	if err != nil {
		errStr := fmt.Sprintf("Error retrieving state from ledger for queried chaincode: %s", err.Error())
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}

	// Compute sum
	sumVal = Aval + Bval

	// Write sumVal back to the ledger
	err = stub.PutState(sum, []byte(strconv.Itoa(sumVal)))
	if err != nil {
		return shim.Error(err.Error())
	}

	fmt.Printf("Invoke chaincode successful. Got sum %d\n", sumVal)
	return shim.Success([]byte(strconv.Itoa(sumVal)))
}

func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	var sum, channelName string // Sum entity
	var Aval, Bval, sumVal int  // value of sum entity - to be computed
	var err error

	if len(args) < 2 {
		return shim.Error("Incorrect number of arguments. Expecting atleast 2")
	}

	chaincodeName := args[0] // Expecting name of the chaincode you would like to call, this name would be given during chaincode install time
	sum = args[1]

	if len(args) > 2 {
		channelName = args[2]
	} else {
		channelName = ""
	}

	// Query chaincode_example02
	f := "query"
	queryArgs := toChaincodeArgs(f, "a")

	//   if chaincode being invoked is on the same channel,
	//   then channel defaults to the current channel and args[2] can be "".
	//   If the chaincode being called is on a different channel,
	//   then you must specify the channel name in args[2]
	response := stub.InvokeChaincode(chaincodeName, queryArgs, channelName)
	if response.Status != shim.OK {
		errStr := fmt.Sprintf("Failed to query chaincode. Got error: %s", response.Payload)
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}
	Aval, err = strconv.Atoi(string(response.Payload))
	if err != nil {
		errStr := fmt.Sprintf("Error retrieving state from ledger for queried chaincode: %s", err.Error())
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}

	queryArgs = toChaincodeArgs(f, "b")
	response = stub.InvokeChaincode(chaincodeName, queryArgs, channelName)
	if response.Status != shim.OK {
		errStr := fmt.Sprintf("Failed to query chaincode. Got error: %s", response.Payload)
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}
	Bval, err = strconv.Atoi(string(response.Payload))
	if err != nil {
		errStr := fmt.Sprintf("Error retrieving state from ledger for queried chaincode: %s", err.Error())
		fmt.Printf(errStr)
		return shim.Error(errStr)
	}

	// Compute sum
	sumVal = Aval + Bval

	fmt.Printf("Query chaincode successful. Got sum %d\n", sumVal)
	jsonResp := "{\"Name\":\"" + sum + "\",\"Value\":\"" + strconv.Itoa(sumVal) + "\"}"
	fmt.Printf("Query Response:%s\n", jsonResp)
	return shim.Success([]byte(strconv.Itoa(sumVal)))
}

func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	function, args := stub.GetFunctionAndParameters()
	if function == "invoke" {
		return t.invoke(stub, args)
	} else if function == "query" {
		return t.query(stub, args)
	}

	return shim.Success([]byte("Invalid invoke function name. Expecting \"invoke\" \"query\""))
}

```

