## cryptogen 详解

[TOC]

证书认证是超级账本中权限管理体系的最基础的机制，而所有证书的生成超级账本提供了一个工具cryptogen用于根据配置文件生成一套证书，并且将这些证书组织为peer和orderer两个核心组件可以直接使用的形式。 

```
usage: cryptogen [<flags>] <command> [<args> ...]

Utility for generating Hyperledger Fabric key material

Flags:
  --help  Show context-sensitive help (also try --help-long and --help-man).

Commands:
  generate [<flags>]
    Generate key material
	--output="crypto-config"  The output directory in which to place artifacts
	--config=CONFIG           The configuration template to use
  showtemplate
    Show the default configuration template

  version
    Show version information

  extend [<flags>]
    Extend existing network
    --input="crypto-config"  The input directory in which existing network place
    --config=CONFIG           The configuration template to use
```

## 一、命令详解

### (1) generate

生成秘钥材料

```
--output="crypto-config"  
		The output directory in which to place artifacts
		生成的一套证书保存的位置
--config=CONFIG
		The configuration template to use
		配置文件的路径
```

### (2) showtemplate

显示一个配置文件的模板。

### (3) version

Show version information

### (4) extend

导出已存在的网络。

```
--input="crypto-config"  
		The input directory in which existing network place
		已存在的网络组件所在的目录
--config=CONFIG           
		The configuration template to use
```



## 二、配置文件解析

配置文件`crypto-config.yaml`：

```yaml
OrdererOrgs:                 #用于orderer 组件使用的组织信息
  - Name: Orderer            #orderermsp的名称
    Domain: example.com      #orderer 的域名后缀
    EnableNodeOUs: true

    CA:                      #CA信息，不用关注，照着填写即可
      Hostname: ca # implicitly ca.org1.example.com
      Country: US
      Province: California
      Locality: San Francisco
      OrganizationalUnit: Hyperledger Fabric
      StreetAddress: address for org # default nil
      PostalCode: postalCode for org # default nil

    Specs: #orderer域名前缀，和Domain一起组成了域名用于tls认证使用
      - Hostname: orderer

PeerOrgs:                     #用于peer 组件使用的组织信息
  - Name: Org1                #组织名称
    Domain: org1.example.com  #组织域名

    CA:
      Country: US
      Province: California
      Locality: San Francisco

    Template:                #组织内参与个体的个数
      Count: 2
    Users:                   #组织内除了管理员用户外的其他用户个数
      Count: 1

  - Name: Org2
    Domain: org2.example.com

    CA:
        Country: US
        Province: California
        Locality: San Francisco
    Template:
      Count: 2
    Users:
      Count: 1
```

## 三、证书分析

[比较全的介绍](https://cloud.tencent.com/developer/article/1337863 )

### 3.1 整体结构

```
 justin@ulord:~/go/exercise/crypto-config$ tree -L 3
.
├── ordererOrganizations
│   └── example.com
│       ├── ca
│       ├── msp
│       ├── orderers
│       ├── tlsca
│       └── users
└── peerOrganizations
    ├── org1.example.com   组织名称
    │   ├── ca             组织的根证书
    │   ├── msp            组织的msp
    │   ├── peers          节点相关证书
    │   ├── tlsca          组织内部的tlsca证书
    │   └── users          组织所属用户
    └── org2.example.com
        ├── ca
        ├── msp
        ├── peers
        ├── tlsca
        └── users
```

`ordererOrganizations/example.com`、`peerOrganizations/org1.example.com`、`peerOrganizations/org2.example.com`这三个目录分别代表了三个机构，

orderer机构一般是虚拟的，或者是超然的，是orderer节点的配置，而每一个机构都有两个根证书，一个用于功能性证书的签发，一个用于tls机制证书的签发，tls机制网上有比较成熟的资料，本文不过多介绍，只关注功能性证书。 

### 3.2 组织结构

以`org1.example.com`组织为例，分析其具体的证书结构

#### ca

存放组织的根证书和对应的私钥文件，默认采用EC算法，证书为自签名。组织内的实体将基于该证书作为证书根。

```
├── ca
│   ├── 64760eacb4a048318e8e09d4a629380ed47564cbb55ef263a6c38367b812f308_sk
│   └── ca.org1.example.com-cert.pem
```

#### tlsca

存放组织tls连接用的根证书和私钥文件。（TLS是传输层安全协议，其实就是SSL，现在叫TLS了）

```
├── tlsca
│   ├── 733f3850314c7ed4f297ffa5cbe4c15e665777f05d3cb72916e10648eb0c097e_sk
│   └── tlsca.org1.example.com-cert.pem
```

#### msp

存放代表该组织的身份信息，只有证书，没有私钥

```
msp
├── admincerts  组织管理员的身份验证证书，被根证书签名。
│   ├── Admin@org1.example.com-cert.pem
│   └── ca.org1.example.com-cert.pem
├── cacerts     组织的根证书，同ca目录下文件。
│   └── ca.org1.example.com-cert.pem
├── config.yaml
└── tlscacerts   用于TLS的ca证书，自签名。
    └── tlsca.org1.example.com-cert.pem
```

#### peers

存放属于该组织的所有peer节点。

```
peers
├── peer0.org1.example.com  第一个peer的信息，包括其msp证书和TLS证书两类。
│   ├── msp
│   │   ├── admincerts 组织管理员的身份验证证书。peer将基于这些证书来认证交易签署这是否为管理员身份。
│   │   │   └── Admin@org1.example.com-cert.pem
│   │   ├── cacerts    组织的根证书.
│   │   │   └── ca.org1.example.com-cert.pem
│   │   ├── keystore   本节点的身份私钥，用来签名。
│   │   │   └── 1c740f7cf63e45890ab9f6204da7db77bf47ff9cd42c7e9d4cd81338af2f3a74_sk
│   │   ├── signcerts  验证本节点签名的证书，被组织根证书签名。
│   │   │   └── peer0.org1.example.com-cert.pem
│   │   └── tlscacerts TLS连接用的身份证书，即组织TLS证书。
│   │       └── tlsca.org1.example.com-cert.pem
│   └── tls 存放tls相关的证书和私钥
│       ├── ca.crt            组织的根证书
│       ├── server.crt        验证本节点签名的证书，被组织根证书签名。
│       └── server.key        本节点的身份私钥，用来签名。
└── peer1.org1.example.com  结构同 peer0.org1.example.com
```

#### users

存放属于该组织的用户的实体。

```
users/
├── Admin@org1.example.com  管理员用户的信息，包括其msp证书和tls证书两类。
│   ├── msp
│   │   ├── admincerts   组织根证书作为管理者身份验证证书。
│   │   │   └── Admin@org1.example.com-cert.pem
│   │   ├── cacerts      组织的根证书.
│   │   │   └── ca.org1.example.com-cert.pem
│   │   ├── keystore     本用户的身份私钥，用来签名。
│   │   │   └── b4998bd9948ed216473d41df3f11f68c6a5801213b5626c23fa7764371c90e9f_sk
│   │   ├── signcerts    管理员用户的身份验证证书，被组织根证书签名。
                         要被某个Peer认可，则必须放到该peer的msp/admincerts下。
│   │   │   └── Admin@org1.example.com-cert.pem
│   │   └── tlscacerts   TLS连接用的身份证书，即组织TLS证书。
│   │       └── tlsca.org1.example.com-cert.pem
│   └── tls
│       ├── ca.crt
│       ├── client.crt
│       └── client.key
└── User1@org1.example.com
```

### 3.3 证书内容

#### 文件类型

查看证书文件（实际上，数字证书就是经过CA认证过的公钥）的标准为X.509，编码格式为pem，以-----BEGIN开头,以-----END结尾。X.509 数字证书不但包括用户名和公共密钥，而且还包括有关该用户的其他信息。除了扩展名为PEM的还有以下这些：

- CRT    应该是certificate的三个字母，还是证书的意思。打开看也是pem编码格式。
- KEY    用来存放一个公钥或私钥，并非X.509证书。打开看依然PEM格式。

证书的默认签名算法为ECDSA，Hash算法为SHA-256。

#### 证书种类

Fabric中设计中考虑了三种类型证书：

- 登记证书（ECert）

  > 颁发给提供了注册凭证的用户或节点实体，长期有效。（主要就是通ECert对实体身份检验）

- 通信证书（TLSCert）

  > TLS证书用来保障通信链路安全，控制对网络层的接入访问，可以对远端实体身份校验，防止窃听。

- 交易证书（TCert）

  > 颁发给用户，控制每个交易的权限，一般针对某个交易，短期有效。（此功能fabric还暂未启用）