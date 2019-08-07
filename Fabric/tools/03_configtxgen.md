## configtxgen 详解

[TOC]

configtxgen工具生成两个内容： Orderer的**bootstrap block**和Fabric的**channel configuration transaction**。 

orderer block是ordering服务的创世区块；

channel transaction文件在create channel时会被广播给orderer。 

```
justin@ulord:~/go/exercise$ configtxgen --help
Usage of configtxgen:
  -asOrg string
        Performs the config generation as a particular organization (by name), only including values in the write set that org (likely) has privilege to set
  -channelID string
        The channel ID to use in the configtx
  -configPath string
        The path containing the configuration to use (if set)
  -inspectBlock string
        Prints the configuration contained in the block at the specified path
  -inspectChannelCreateTx string
        Prints the configuration contained in the transaction at the specified path
  -outputAnchorPeersUpdate string
        Creates an config update to update an anchor peer (works only with the default channel creation, and only for the first update)
  -outputBlock string
        The path to write the genesis block to (if set)
  -outputCreateChannelTx string
        The path to write a channel creation configtx to (if set)
  -printOrg string
        Prints the definition of an organization as JSON. (useful for adding an org to a channel manually)
  -profile string
        The profile from configtx.yaml to use for generation. (default "SampleInsecureSolo")
  -version
        Show version information
```

## 一、命令参数

### (1) asOrg

Performs the config generation as a particular organization (by name), only including values in the write set that org (likely) has privilege to set

作为特定的组织(按名称)执行配置生成，只包括org(可能)有权设置的写集中的值

### (2) channelID

The channel ID to use in the configtx

configtx中使用的通道ID

### (3) profile 

The profile from `configtx.yaml` to use for generation. (default "SampleInsecureSolo")

从配置文件`configtx.yaml`中的profile字段下的读取配置（默认选取`SampleInsecureSolo`）

- ``../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME``

  ```
  [configtxgen] main -> INFO 001 Loading configuration
  [configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/justin/go/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
  [configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
  [configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /home/justin/go/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
  [configtxgen] doOutputChannelCreateTx -> INFO 005 Generating new channel configtx
  [configtxgen] doOutputChannelCreateTx -> INFO 006 Writing new channel tx
  ```

### (4) outputBlock

The path to write the genesis block to (if set)

设置生成的创世块存放的路径（如果有的话）

### (5) outputCreateChannelTx

The path to write a channel creation configtx to

将创建的 通道配置文件 存放的路径。（如果有的话）

### (6) outputAnchorPeersUpdate

Creates an config update to update an anchor peer (works only with the default channel creation, and only for the first update)

创建一个更新配置文件，用于锚节点的更新（只适用于默认的通道创建，也只适用于第一次更新）。

### (7) printOrg string

Prints the definition of an organization as JSON. (useful for adding an org to a channel manually)

将组织的定义数据输出为JSON。（用于手动向通道添加组织）

### (8) configPath string

The path containing the configuration to use (if set)

包含要使用的配置的路径

### (9)  inspectBlock string

Prints the configuration contained in the block at the specified path

按指定路径打印块中包含的配置

### (10) inspectChannelCreateTx string

Prints the configuration contained in the transaction at the specified path

按指定路径打印交易中包含的配置

## 二、配置文件详解

### (1) Organizations

定义了不同的组织标识，在后续的配置中引用。

- `Name` 
- `ID`     用于加载MSP定义的ID
- `MSPDir`  包含MSP配置的文件系统路径
- `Policies`  协议，策略树--组织级别。 
  - 路径：`/Channel/<Application|Orderer>/<OrgName>/<PolicyName>`
  - `Rule` 
    - 订单： `member`和`admin`
    - 组织：`admin`,`peer`,`client`
- `AnchorPeers` 定义一个组织的对外通信节点（锚节点），在非Orderer中才需要定义。

```yaml
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/example.com/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp

        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"

        AnchorPeers:
            # AnchorPeers定义了可用于跨组织Gossip通信的节点的位置。 
            # 注意，此值仅在Application部分上下文的genesis块中编码
            - Host: peer0.org1.example.com
              Port: 7051
```

### (2) Capabilities

本节定义fabric network的功能。这是v1.1.0的一个新概念，不应该在v1.0的混合网络中使用。
x个对等点和顺序。功能定义了fabric二进制文件中必须提供的特性，以便该二进制文件安全地参与
fabric网络。例如，如果添加了新的MSP类型，较新的二进制文件可能会识别并验证来自该类型
的签名，而没有此支持的较老的二进制文件将无法验证这些事务。这可能导致不同版本的fabric
二进制文件具有不同的世界状态。

简单来说，使用新版本的特性时，注意不同版本之间的不兼容的更新。

```yaml
Capabilities:
    # Channel功能适用于订购者和同行，并且必须得到两者的支持。
    # 将功能的值设置为true以使其需要。
    Channel: &ChannelCapabilities
        # Channel的V1.3是行为的一个标记，已被确定为运行在v1.3.x级别的所有orderers
        # 和peer所需的行为，但是与先前版本的orderers和peer不兼容。
        # 在启用V1.3通道功能之前，请确保通道上的所有订购者和对等方均为v1.3.0或更高版本。
        V1_3: true

    # Orderer功能仅适用于订货人，并且可以安全地与之前的版本同行一起使用。
    Orderer: &OrdererCapabilities
        # V1.1 版本与之前的不兼容，v1.1之后的版本都可用使用
        V1_1: true

    # 应用程序功能仅适用于对等网络，可以安全地与先前的发布订阅者一起使用。
    Application: &ApplicationCapabilities
        # V1.3 for Application启用了Fabric v1.3的新的非向后兼容功能和修复。
        V1_3: true
        # 启用了Fabric v1.2的新的非向后兼容功能和修复 
        V1_2: false
        # 启用了Fabric v1.1的新的非向后兼容功能和修复 
        # (注意，如果设置了更高版本的功能，则无需设置此功能)
        V1_1: false
```

### (3) Application

APP 相关参数的配置事务或创建块的值

- `Organizations`
- `Policies`
- `Capabilities`

```yaml
Application: &ApplicationDefaults

    # 组织是在网络的应用程序端定义为参与者的组织列表
    Organizations:

    #   /Channel/Application/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    # 功能描述了应用程序级功能，有关完整说明，请参阅此文件中其他位置的专用功能部分
    Capabilities:
        <<: *ApplicationCapabilities
```

### (4) Orderer

此部分定义要编码为orderer相关参数的config事务或genesis块的值

```yaml
Orderer: &OrdererDefaults

    # 要使用的排序类型，可选 "solo" 和 "kafka"
    OrdererType: solo

    # 这里的地址是同行和客户可以连接到的订单的非穷尽列表。 
    # 在此列表中添加/删除节点不会影响他们参与订购。
    # 注意：在个人情况下，这应该是一个项目列表。
    Addresses:
        - orderer.example.com:7050

    # 创建批处理之前要等待的时间
    BatchTimeout: 2s

    # 控制成块的消息数量
    BatchSize:

        # 批处理中允许的最大消息数。
        MaxMessageCount: 10

        # Absolute Max Bytes: 批处理中序列化消息允许的绝对最大字节数。
        AbsoluteMaxBytes: 99 MB

        # Preferred Max Bytes: 批处理中允许序列化消息的首选最大字节数。
        # 大于首选最大字节的消息将导致批处理大于首选最大字节。
        PreferredMaxBytes: 512 KB

    Kafka:
        # Brokers: A list of Kafka brokers to which the orderer connects
        # NOTE: Use IP:port notation
        Brokers:
            - 127.0.0.1:9092

    # Organizations 是在网络的订货人一侧被定义为参与者的组织列表
    Organizations:

    # 策略(Policies)在配置树的这个级别定义策略集
    # For Orderer policies, their canonical path is
    #   /Channel/Orderer/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

        # 块验证指定必须包含来自orderer的哪些签名，以便对等方对其进行验证。
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"
```

### (5) Channel

通道相关参数的配置事务或创建块的值。

```yaml
Channel: &ChannelDefaults
    #   /Channel/<PolicyName>
    Policies:
        # Who may invoke the 'Deliver' API
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        # Who may invoke the 'Broadcast' API
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        # By default, who may modify elements at this config level
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    Capabilities:
        <<: *ChannelCapabilities
```



### (6) Profiles ☆

```yaml
Profiles:

    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
            Capabilities:
                <<: *ApplicationCapabilities

    SampleDevModeKafka:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: kafka
            Kafka:
                Brokers:
                - kafka.example.com:9092

            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                - *Org1;17H- *Org2
```

