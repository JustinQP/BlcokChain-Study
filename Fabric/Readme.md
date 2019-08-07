### 发展史

2015 年 12 月，开源世界的旗舰组织 —— Linux 基金会 牵头，联合 30 家初始企业成员（包 括 IBM、Accenture、Intel、J.P.Morgan、R3、DAH、DTCC、FUJITSU、HITACHI、 SWIFT、Cisco 等），共同 宣布 了超级账本（Hyperledger）联合项目的成立。超级账本项目 致力为透明、公开、去中心化的企业级分布式账本技术提供开源参考实现，并推动区块链和 分布式账本相关协议、规范和标准的发展。项目官方网站为 hyperledger.org。 成立之初，项目就收到了众多开源技术贡献。IBM 贡献了 4 万多行已有的 Open Blockchain 代码，Digital Asset 贡献了企业和开发者相关资源，R3 贡献了新的金融交易架构，Intel 贡献 了分布式账本相关的代码。 

作为一个联合项目（Collaborative Project），超级账本由面向不同目的和场景的子项目构 成。目前包括 **Fabric、SawToothLake、Iroha、Blockchain Explorer、Cello、Indy、 Composer、Burrow、Quilt、Caliper、Ursa、Grid** 等 12 大顶级项目，所有项目都遵守 Apache v2 许可，并约定共同遵守如下的 基本原则：

- 重视模块化设计：包括交易、合同、一致性、身份、存储等技术场景； 

- 重视代码可读性：保障新功能和模块都可以很容易添加和扩展； 

- 可持续的演化路线：随着需求的深入和更多的应用场景，不断增加和演化新的项目。 



## 主要项目

**Fabric**：包括 Fabric、Fabric CA、Fabric SDK（包括 Node.Js、Java、Python 和 Go 语 言）等，目标是区块链的基础核心平台，支持 PBFT 等新的共识机制，支持权限管理， 最早由 IBM 和 DAH 于 2015 年底发起； 

**Sawtooth**：包括 arcade、core、dev-tools、validator、mktplace 等。是 Intel 主要发起 和贡献的区块链平台，支持全新的基于硬件芯片的共识机制 Proof of Elapsed Time（PoET）。 2016 年 4 月贡献到社区。

**Blockchain Explorer**：提供 Web 操作界面，通过界面快速查看查询绑定区块链的状态 （区块个数、交易历史）信息等，由 DTCC、IBM、Intel 等开发支持。2016 年 8 月贡献 到社区。 

**Iroha**：账本平台项目，基于 C++ 实现，带有不少面向 Web 和 Mobile 的特性，主要由 Soramitsu 于 2016 年 10 月发起和贡献。 

**Cello**：提供区块链平台的部署和运行时管理功能。使用 Cello，管理员可以轻松部署和管 理多条区块链；应用开发者可以无需关心如何搭建和维护区块链，由 IBM 团队于 2017 年 1 月贡献到社区。 

**Indy**：提供基于分布式账本技术的数字身份管理机制，由 Sovrin 基金会发起，2017 年 3 月底正式贡献到社区。 

**Composer**：提供面向链码开发的高级语言支持，自动生成链码代码等，由 IBM 团队发 起并维护，2017 年 3 月底贡献到社区。 

**Burrow**：提供以太坊虚拟机的支持，实现支持高效交易的带权限的区块链平台，由 Monax 公司发起支持，2017 年 4 月贡献到社区。 

**Quilt**：对 W3C 支持的跨账本协议 Interledger 的 Java 实现。2017 年 10 月正式贡献到 社区。 

**Caliper**：提供对区块链平台性能的测试工具，由华为 公司发起支持。2018 年 3 月正式 贡献到社区。 

**Ursa**：提供一套密码学相关组件，初始贡献者包括来自 Fujitsu、Sovrin、Intel、 DFINITY、State Street、IBM、Bitwise IO 等企业的开发者。2018 年 11 月正式被接收到 社区。

 **Grid**：提供帮助快速构建供应链应用的框架，由 Cargill、Intel 和 Bitwise IO 公司发起支 持。2018 年 12 月正式贡献到社区。 



## 资料

 [Hyperledger](<https://www.hyperledger.org/> )      [Hyperledger Fabric](<https://www.hyperledger.org/projects/fabric> )

[Hyperledger 英文文档](<https://hyperledger-fabric.readthedocs.io/en/master/whatis.html> )    [Hyperledger 中文文档](<https://hyperledgercn.github.io/hyperledgerDocs/> )



架构

- [Hyperledger Fabric1.0架构概览](<https://www.8btc.com/article/117316> )
- [Hyperledger Fabric架构详解](<https://www.cnblogs.com/earvin/p/10025902.html> )  介绍一些核心概念的技术细节
- [Fabric和Sawtooth技术分析（上）](https://www.cnblogs.com/earvin/p/10939348.html)   Fabric框架，工作流程讲的很清楚

入门篇

- [Hyperledger Fabric V1.0– 开发者快速入门](<https://zhuanlan.zhihu.com/p/25070745> )

技术栈

- [fabric原理之锚节点](https://www.cnblogs.com/StephenWu/p/10485739.html)
- 链码
  - [Fabric--链码的概念与使用](<https://blog.csdn.net/qq_37133717/article/details/80932904> )
  - 
- 技术
  - Gossip
    - [Gossip 数据传输协议](https://blog.csdn.net/baidu_39649815/article/details/76468315)
    - [[蜗牛讲-Fabric入门之基于Gossip的数据传播概述](https://www.cnblogs.com/StephenWu/p/9045304.html)
  - MSP
    - [蜗牛讲-fabric原理之MSP介绍](https://www.cnblogs.com/StephenWu/p/10485713.html)
    - [蜗牛讲-fabric原理之证书生成](https://www.cnblogs.com/StephenWu/p/9130438.html)
    - [Fabric证书解析](https://www.cnblogs.com/earvin/p/9546504.html)
    - [Membership Service Providers (MSP)](https://blog.csdn.net/baidu_39649815/article/details/76468249 )  证书原理，有点深奥的样子。一遍看不懂。todo
    - [超级账本之权限管理浅析](https://cloud.tencent.com/developer/article/1337863 )
  - [TLS传输层安全协议](https://blog.csdn.net/fengzei886/article/details/52352183)
  - gRPC
  - [Docker-Compose详解](https://blog.csdn.net/u011781521/article/details/80464826)
  - levelDB

博客站点

- [一个有关Hyperledger的有趣的站点 ](<https://hyperledgerstore.com/> )
- [一套fabric入门级博客](https://www.cnblogs.com/llongst/tag/fabric/)

书

- [区块链技术指南](<https://yeasy.gitbooks.io/blockchain_guide/content/> )
- [Hyperledger 源码分析之 Fabric (残)](<https://yeasy.gitbooks.io/hyperledger_code_fabric/content/protos/> )
