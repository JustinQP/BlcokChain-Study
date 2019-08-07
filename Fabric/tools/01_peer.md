## peer 命令详解

[TOC]



```bash
justin@ubuntu:~/go/src/github.com/fabric-samples/bin$ peer --help
Usage:
  peer [command]

Available Commands:
  chaincode   Operate a chaincode: install|instantiate|invoke|package
              |query|signpackage|upgrade|list.
  channel     Operate a channel: create|fetch|join|list|update|signconfigtx|getinfo.
  logging     Logging configuration: getlevel|setlevel|getlogspec|setlogspec|revertlevels.
  node        Operate a peer node: start|status.

Flags:
  -h, --help   help for peer

Use "peer [command] --help" for more information about a command.
```



#### 通用的参数

---

```
-C, --channelID string               
  			The channel on which this command should be executed
  			应该执行此命令的通道
-c, --ctor string                    
             Constructor message for the chaincode in JSON format (default "{}")
             用JSON格式构造链码的消息（默认为“{}”）
-l, --lang string                    
             Language the chaincode is written in (default "golang")
             编写链码的语言（默认为“golang”）
-n, --name string                    
             Name of the chaincode.  
             链码的名称
-h, --help   
			help for command
			命令的详细帮助信息
    --connectionProfile string      
             Connection profile that provides the necessary connection information 
             for the network. Note: currently only supported for providing peer 
             connection information
             连接配置文件，为网络提供必要的连接信息。注意：目前仅支持提供对等连接信息
    --peerAddresses stringArray      
             The addresses of the peers to connect to
             要连接的对等节点的地址
    --tlsRootCertFiles stringArray   
             If TLS is enabled, the paths to the TLS root cert files of the peers 
             to connect to. The order and number of certs specified should match 
             the --peerAddresses flag
             如果启用了TLS，则指向要连接的对等方的TLS根证书文件的路径。
             指定的证书的顺序和数量应与--peadAddresses标志匹配
```

#### 全局参数

```
Flags:
      --cafile string                       
			Path to file containing PEM-encoded trusted certificate(s) 
			for the ordering endpoint
			包含订购端点的PEM编码可信证书的文件路径
      --certfile string                     
			Path to file containing PEM-encoded X509 public key to use 
			for mutual TLS communication with the orderer endpoint
			包含PEM编码的X509公钥的文件路径，用于与orderer端点进行相互TLS通信
      --clientauth                          
			Use mutual TLS when communicating with the orderer endpoint
			与orderer端点通信时使用相互TLS
      --connTimeout duration                
			Timeout for client to connect (default 3s)
			客户端连接超时（默认3秒）
      --keyfile string                      
			Path to file containing PEM-encoded private key to use 
			for mutual TLS communication with the orderer endpoint
			包含PEM编码私钥的文件路径，用于与orderer端点进行相互TLS通信
	  -o, --orderer string                      
			Ordering service endpoint
			订购服务端点
      --ordererTLSHostnameOverride string   
			The hostname override to use when validating the TLS connection to the orderer.
			验证与订货人的TLS连接时使用的主机名覆盖。
      --tls                                 
			Use TLS when communicating with the orderer endpoint
			与orderer端点通信时使用TLS
      --transient string                    
			Transient map of arguments in JSON encoding
			JSON编码中的参数的瞬态映射
```



### (1) chaincode

```
Operate a chaincode: install|instantiate|invoke|package|query|signpackage|upgrade|list.

Usage:
  peer chaincode [command]

Available Commands:
  install     Package the specified chaincode into a deployment spec 
              and save it on the peer's path.
  instantiate Deploy the specified chaincode to the network.
  invoke      Invoke the specified chaincode.
  list        Get the instantiated chaincodes on a channel or installed chaincodes on a peer.
  package     Package the specified chaincode into a deployment spec.
  query       Query using the specified chaincode.
  signpackage Sign the specified chaincode package
  upgrade     Upgrade chaincode.

Use "peer chaincode [command] --help" for more information about a command.
```

#### install

将指定的链码打包到部署规范中，并将其保存在节点的路径中。

```
-c,-h,-l,-n,--connectionProfile,--tlsRootCertFiles
  
-p, --path string                    
		Path to chaincode       
		链码的路径
-v, --version string                 
		Version of the chaincode specified in install/instantiate/upgrade commands
		install / instantiate / upgrade命令中指定的链代码版本
```

#### instantiate

Deploy the specified chaincode to the network.

将指定的链码部署到网络。 

```
-C,-c,-h,-l,-n,--connectionProfile,--peerAddresses,--tlsRootCertFiles
--collections-config string      
		The fully qualified path to the collection JSON file including the file name.
		集合JSON文件的完全限定路径，包括文件名
-E, --escc string                    
		The name of the endorsement system chaincode to be used for this chaincode
  		用于此链码的背书系统链码的名称
-P, --policy string                  
  		The endorsement policy associated to this chaincode
  		与此链码相关的背书协议
-v, --version string                 
  		Version of the chaincode specified in install/instantiate/upgrade commands
  		install / instantiate / upgrade命令中指定的链代码版本
-V, --vscc string                    
  		The name of the verification system chaincode to be used for this
		用于验证系统链码的名称
```

#### invoke
invoke 执行，

```
Flags:
  -C,-c,-h,-n,--connectionProfile,--peerAddresses,-- tlsRootCertFiles
      --waitForEvent                  
			Whether to wait for the event from each peer's deliver filtered service 
			signifying that the 'invoke' transaction has been committed successfully
			是否等待来自每个对等方的传递筛选服务的事件，该事件表示“invoke”事务已成功提交
      --waitForEventTimeout duration   
			Time to wait for the event from each peer's deliver filtered service 
			signifying that the 'invoke' transaction has been committed 
			successfully (default 30s)
			等待每个对等方提供过滤服务的事件的时间，表示“调用”事务已成功提交（默认为30秒）
```

#### list

如果指定通道，则在通道中获取实例化的链码，或者在对等点上安装链码   

```
-C,-h, --connectionProfile,--peerAddresses,--tlsRootCertFiles                                 

--installed                     
		Get the installed chaincodes on a peer
		获取对等点上已安装的链码
--instantiated                   
		Get the instantiated chaincodes on a channel
		获取通道上的实例化链码
```

#### package

Package the specified chaincode into a deployment spec. 

将指定的链代码打包到部署规范中。

```
-c, -l, -h, -n

-s, --cc-package                  
		create CC deployment spec for owner endorsements instead of raw CC deployment spec
		为所有者背书创建CC部署规范，而不是原始的CC部署规范
-i, --instantiate-policy string   
		instantiation policy for the chaincode
		链代码的实例化策略
-p, --path string                 
		Path to chaincode
-S, --sign                        
		if creating CC deployment spec package for owner endorsements, 
		also sign it with local MSP
		如果为所有者背书创建CC部署规范包，也要与本地MSP签署
-v, --version string           
		Version of the chaincode specified in install/instantiate/upgrade commands
```

#### query

Get endorsed result of chaincode function call and print it. It won't generate transaction.

获取链码函数调用的认可结果并打印出来。它不会生成交易。

```
Flags:
-C, -c, -h, -n, --peerAddresses, --tlsRootCertFiles
-x, --hex
		If true, output the query value byte array in hexadecimal. Incompatible with --raw
		如果为真，则以十六进制输出查询值字节数组。不兼容--raw
-r, --raw                            
		If true, output the query value as raw bytes, otherwise format as a printable string 
		如果为真，则输出原始字节的查询值，否则格式化为可打印字符串
```



#### signpackage

Sign the specified chaincode package

签署指定的链码包 

```
peer chaincode signpackage [flags]
```

#### upgrade

Upgrade an existing chaincode with the specified one. The new chaincode will immediately replace the existing chaincode upon the transaction committed.

用指定的链码升级现有的链码。提交事务后，新的链代码将立即替换现有的链代码。 

```
-C, -c, -h, -l, -n,  --connectionProfile,--peerAddresses, --tlsRootCertFiles

--collections-config string      
		The fully qualified path to the collection JSON file including the file name
		集合JSON文件的完全限定路径，包括文件名
-p, --path string
		Path to chaincode
-P, --policy string
		The endorsement policy associated to this chaincode
		与此链码关联的背书策略
-v, --version string
		Version of the chaincode specified in install/instantiate/upgrade commands
-V, --vscc string
		The name of the verification system chaincode to be used for this chaincode
		用于此链码的验证系统链码的名称
```

### (2) channel

Operate a channel: create|fetch|join|list|update|signconfigtx|getinfo.

```
justin@ulord:~$ peer channel --help
Operate a channel: create|fetch|join|list|update|signconfigtx|getinfo.

Usage:
  peer channel [command]

Available Commands:
  create       Create a channel
  fetch        Fetch a block
  getinfo      get blockchain information of a specified channel.
  join         Joins the peer to a channel.
  list         List of channels peer has joined.
  signconfigtx Signs a configtx update.
  update       Send a configtx update.
```

#### create

Create a channel and write the genesis block to a file.

创建通道并将genesis块写入文件。 

```
-c, --channelID string
		In case of a newChain command, the channel ID to create. It must be all lower case, 
		less than 250 characters long and match the regular expression: [a-z][a-z0-9.-]*
		对于newChain命令，要创建的通道ID。它必须全是小写，长度小于250个字符，
		并且匹配正则表达式:[a-z][a-z0-9.-]*
-f, --file string
		Configuration transaction file generated by a tool such as configtxgen for 
		submitting to orderer
		由configtxgen之类的工具生成的配置事务文件，用于提交给orderer
--, --outputBlock string
    	The path to write the genesis block for the channel. (default ./<channelID>.block)
    	为通道编写genesis块的路径。
-t, --timeout duration
		Channel creation timeout (default 5s)
		通道创建超时
```

- `peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`

  ```
  [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  [cli.common] readBlock -> INFO 002 Received block: 0
  ```

#### fetch

Fetch a specified block, writing it to a file.

获取指定的块，并将其写入文件。 

```
peer channel fetch <newest|oldest|config|(number)> [outputfile] [flags]

-c, --channelID string
```

- `peer channel fetch newest -c mychannel`

  ```
  [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  [cli.common] readBlock -> INFO 002 Received block: 6
  ```

  当前目录下多了一个 `mychannel_newest.block`的文件

- `peer channel fetch config -c mychannel`

  ```
  [cli.common] readBlock -> INFO 002 Received block: 6
  [cli.common] readBlock -> INFO 003 Received block: 2
  ```

  生成的块数据文件，默认放在当前目录下。

#### getinfo

get blockchain information of a specified channel. Requires '-c'.

获取指定通道的区块链信息。 Requires '-c'.

```
peer channel getinfo [flags]

-c, --channelID string
```

- `peer channel getinfo -c mychannel`

  ```
  Blockchain info: {"height":6,"currentBlockHash":"3WXFNm1NazzMlT4eSUfZGFrxuZKb8/TK/VtDg6yhz8U=","previousBlockHash":"KP20sPqasvkejG1PcJk9qzcL2vDAXIeS2BtJaoeOPv0="}
  ```

#### join

Joins the peer to a channel.

将节点连接到通道。 

```
-b, --blockpath string   Path to file containing genesis block
```

- `peer channel join -b mychannel.block`

  ```
  // 成功创建输出：
  [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
  
  // 失败创建输出：
  Error: proposal failed (err: bad proposal response 500: cannot create ledger from genesis block: LedgerID already exists)
  ```

#### list

List of channels peer has joined.

列举出节点已加入的通道

- `peer channel list`

  ```
  Channels peers has joined:
  mychannel
  ```

#### signconfigtx

Signs the supplied configtx update file in place on the filesystem. Requires '-f'.

在文件系统上对提供的configtx更新文件进行签名。 Requires '-f'.

```
peer channel signconfigtx [flags]

-f, --file string   
		Configuration transaction file generated by a tool such as configtxgen for submitting to orderer
		由configtxgen之类的工具生成的配置事务文件，用于提交给orderer
```

#### update

Signs and sends the supplied configtx update file to the channel. Requires '-f', '-o', '-c'.

签名并将提供的configtx更新文件发送到通道  Requires '-f', '-o', '-c'.

```
-c, --channelID string
-f, --file string        
		Configuration transaction file generated by a tool such as configtxgen for 
		submitting to orderer
		由configtxgen之类的工具生成的交易配置文件，用于提交给orderer
-o, --orderer string                      
		Ordering service endpoint
		订单服务端点
```

- `peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`

  ```
  [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
  [channelCmd] update -> INFO 002 Successfully submitted channel update
  ```

  

### (3) logging

Logging configuration: getlevel|setlevel|getlogspec|setlogspec|revertlevels.

```
Usage:
  peer logging [command]

Available Commands:
  getlevel     Returns the logging level of the requested logger.
  			  peer logging getlevel <logger> [flags]
  			  返回请求的日志程序的日志级别。注意:日志程序名称应该与日志中显示的名称完全匹配。 
  			  
  getlogspec   Returns the active log spec.
  			  返回对等点的活动日志规范 
  		       
  revertlevels Reverts the logging spec to the peer's spec at startup.
  			  在启动时将日志记录格式还原成默认格式。
  				
  setlevel     Adds the logger and log level to the current logging spec.
  			  peer logging setlevel <logger> <log level> [flags]
  			  将日志记录器和日志级别添加到当前日志规范中。
  			  
  setlogspec   Sets the logging spec.
```

### (4) node

```
Operate a peer node: start|status.

Usage:
  peer node [command]

Available Commands:
  start       Starts the node.
  status      Returns status of the node.
```

#### start

Starts a node that interacts with the network.

启动与网络交互的节点。 

```
--peer-chaincodedev   
	Whether peer in chaincode development mode
	是否处于链码开发模式
```

#### status

Returns the status of the running node.

返回运行节点的状态。 

- `peer node status`

  ```
  status:STARTED
  ```