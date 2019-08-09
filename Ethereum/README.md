中文是有版本的。英文的都是指向最新的官方文档

[TOC]

## 语言

### Solidity

[Solidity](https://solidity.readthedocs.io/en/latest/) 一门专门用来写以太坊智能合约的语言。被多条公链支持。

[Solidity 0.5.9 中文](https://learnblockchain.cn/docs/solidity/index.html) （2019/05）



### Vyper

[Vyper](https://vyper.readthedocs.io/en/latest/)是一种通用的、实验性的编程语言,Vyper在逻辑上类似于Solidity，在语法上类似于Python.



### Remix-ide

[Remix](https://remix.ethereum.org/) 在线IDE，用于编译，测试，部署，调试以太坊的合约，支持solidity和vyper.



## 框架

### truffle

Truffle是一个世界级的开发环境，测试框架，以太坊的资源管理通道，致力于让以太坊上的开发变得简单。现在共有3个工具:

[truffle](https://www.trufflesuite.com/docs/truffle/overview)  用于开发，测试，部署以太坊智能合约。

[gannache](https://www.trufflesuite.com/docs/ganache/overview)  用于本地搭建一个以太坊的私有链，用于开发测试。

[drizzle](https://www.trufflesuite.com/docs/drizzle/overview)  是一个`Dapp`开发的框架，包括前后端。

[Truffle 5.0.0 中文](https://learnblockchain.cn/docs/truffle/index.html) （2019/05） 



### web3

[web3.js](https://github.com/ethereum/wiki/wiki/JavaScript-API) 是一套和以太坊节点进行通信的API，如果我们需要基于以太坊来开发去中心化应用，就可能需要使用web3（或者使用 `ethers.js`），例如需要通过Web3来获取节点状态，获取账号信息，调用合约、监听合约事件等等。

[web3.js 0.20.x 中文](https://learnblockchain.cn/docs/web3js-0.2x/) （2019/05）

[Python Web3.py](https://github.com/ethereum/web3.py)  [英文文档](https://web3py.readthedocs.io/en/stable/)

[Java web3j](https://github.com/web3j/web3j)



### ethers

[ethers.js](https://docs.ethers.io/ethers.js/html/) 库旨在为以太坊区块链及其生态系统提供一个小而完整的 JavaScript API 库 它最初是与 [ethers.io](https://ethers.io/) 一起使用，现在已经扩展为更通用的库。

[ethers.js 4.0 中文](https://learnblockchain.cn/docs/ethers.js/)  （2019/05 ）



### openzeppelin

[OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) 一个想要打造一个安全合约代码库的项目。很活跃，有很多合约示例代码。以太坊合约开发必备。



## 连接服务

### Metamask

MetaMask是一款在浏览器上使用的插件类型的以太坊钱包，该钱包不需要下载，只需要在谷歌浏览器添加对应的扩展程序即可，非常轻量级，使用起来也非常方便。

### Infura

[Infura](https://infura.io/) 是一个托管的以太坊节点集群，可以将你开发的以太坊智能合约部署到infura提供的节点上，而无需搭建自己的以太坊节点。



### Etherscan

[Etherscan](https://etherscan.io) 是以太坊上应用最广泛的区块链浏览器，也提供 [API 服务](https://etherscan.io/apis)。 我们知道以太坊节点提供的API功能有限，尤其是需要一些多个区块相关的数据时，必须要依靠`Etherscan API`这样的服务。Etherscan API是社区提供的服务，仅支持每秒 5 个GET或POST请求，可以在这个地址 [API-Keys](https://etherscan.io/myapikey) 申请一个Key。(etherscan经常被墙，不知道什么时候能出来)

[Etherscan API 中文](https://learnblockchain.cn/docs/etherscan/)  （2019/05）



## 其他

### 安全技术

作为合约开发者。合约的安全至关重要。推荐几个设计智能合约安全类型的网站。

[Paper 区块链专栏](https://paper.seebug.org/category/blockchain/)  一个专门介绍安全相关知识的技术型博客网站。

[区块链安全信息平台 - 白帽汇安全研究院](https://www.bcsec.org/)  专注区块链安全的网站。

[安全客](https://www.anquanke.com/)   进去看看就知道了。有部分是介绍区块链安全的。

[国家信息安全漏洞库](http://www.cnnvd.org.cn/index.html)  一看这名字。你懂的  -，-



### 反编译

[在线反编译网站](https://ethervm.io/decompile#func_profit)











