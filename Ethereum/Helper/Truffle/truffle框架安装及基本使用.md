### 安装环境

直接命令 `npm install -g truffle`

直接依赖 npm,node, Ubuntu环境下直接安装 node,npm并升级到最新版本

```
sudo apt-get install nodejs npm && \ 
sudo npm install n -g && \
sudo n stable && \
sudo npm install npm -g
```

Windows 环境 npm,node安装[教程](http://jaychenfe.leanote.com/post/windows-上优雅的安装-node-和-npm)



### 部署命令

初次部署： `truffle migrate`

重新部署：`truffle migrate --reset`



### 部署文件

​	先修改`./migrations/2_initial_migration.js`

​	注意，文件名的前缀是数字，后缀是描述。为了记录迁移是否成功运行，需要有编号的前缀。后缀是纯粹为了人类的可读性和理解力。



### 1.基本配置

#### artifacts.require()

​	在迁移开始时，我们通过artifacts.require（）方法告诉Truffle我们想与哪些合同进行交互。 这个方法类似于Node的要求，但在我们的例子中，它特别返回了我们可以在其余部署脚本中使用的抽象合约。 指定的名称应与该源文件中的合同定义的名称匹配。 不传递源文件的名称，因为文件可以包含多个合同。举例：

```
// fileName: ./contracts/Contracts.sol
contract ContractOne { // ... }

contract ContractTwo { // ... }
```

如果只用合约中的`ContractTwo`你的`artifacts.require()`这样写：

```
var ContractTwo = artifacts.require("ContractTwo");
```

如果两者都要用：

```
var ContractOne = artifacts.require("ContractOne");
var ContractTwo = artifacts.require("ContractTwo");
```



#### module.exports

​	所有迁移都必须通过module.exports语法导出函数。 每次迁移导出的函数都应接受部署者对象作为其第一个参数。 此对象通过为部署智能合约提供清晰的语法以及执行某些部署更普通的职责（例如保存已部署的工件以供以后使用）来帮助部署。 部署者对象是用于暂存部署任务的主要界面，其API在本页底部描述。



#### Initial migration

​	Truffle 框架中必须保留 `contracts/Migrations.sol` 和 `migrations/1_initial_migration.js`这两个文件，部署的时候，将先部署这个合约。



### 2.部署器(Deployer)

​	您的部署文件将使用部署器来完成部署任务。 因此，您可以同步编写部署任务，它们将以正确的顺序执行：

```
// 先部署A再部署B,两者直接无直接联系
deployer.deploy(A);
deployer.deploy(B);
```

```
// 部署A后把A的地址作为参数再部署B
deployer.deploy(A).then(function() {
  return deployer.deploy(B, A.address);
});
```

后面有详细的API文档说明



### 3.网络配置

可以根据部署到的网络有条件地运行部署步骤。 这是一项高级功能，因此请在继续之前先查看“Networks”部分。

要有条件地暂存部署步骤，请编写迁移，以便它们接受第二个参数，称为Network。 例：

```
module.exports = function(deployer, network) {
  if (network == "live") {
    // Do something specific to the network named "live".
  } else {
    // Perform a different step otherwise.
  }
}
```



### 4.选择账户

​	迁移还传递了Ethereum客户端和web3提供商提供给您的帐户列表，供您在部署期间使用。这是从`web3.eth.getAccounts()`返回的相同的帐户列表。

```
module.exports = function(deployer, network, accounts) {
  // Use the accounts within your migrations.
}
```



### 5.Deployer.API

部署程序包含许多可用于简化迁移的功能。

#### deployer.deploy

```
deployer.deploy(contract, args..., options)
```

使用可选的构造函数参数部署由合同对象指定的特定合同。这对单例合约很有用，因此dapp只存在此合约的一个实例。这将在部署后设置合同的地址（即，Contract.address将等于新部署的地址），并且它将覆盖存储的任何先前地址。

您可以选择传递一组合同或一组数组，以加快多个合同的部署。此外，**最后一个参数是一个可选对象**，可以包含名为overwrite的键以及其他事务参数，例如gas和from。如果**overwrite**设置为false，则部署者将不会部署此合同（如果已经部署了该合同）。这对于由外部依赖项提供合同地址的某些情况很有用。

请注意，在调用deploy之前，您需要首先部署和链接合同所依赖的任何库。有关详细信息，请参阅下面的链接功能。

有关更多信息，请参文档



```
// 在没有构造函数参数的情况下部署单个合约
deployer.deploy(A);

// 使用构造函数参数部署单个合同
deployer.deploy(A, arg1, arg2, ...);

// 如果已经部署了此合同，请不要部署它
deployer.deploy(A, {overwrite: false});

//为部署设置最大Gas 和 “from” 地址
deployer.deploy(A, {gas: 4612388, from: "0x...."});

// Deploy multiple contracts, some with arguments and some without.
// This is quicker than writing three `deployer.deploy()` statements as the deployer
// can perform the deployment as a single batched request.
deployer.deploy([
  [A, arg1, arg2, ...],
  B,
  [C, arg1]
]);

// External dependency example:
//
// For this example, our dependency provides an address when we're deploying to the
// live network, but not for any other networks like testing and development.
// When we're deploying to the live network we want it to use that address, but in
// testing and development we need to deploy a version of our own. Instead of writing
// a bunch of conditionals, we can simply use the `overwrite` key.
deployer.deploy(SomeDependency, {overwrite: false});
```



#### deployer.link

```
deployer.link(library, destinations)
```

​	将已部署的库链接到合同或多个合同。 目的地可以是单个合同或多个合同的数组。 如果目的地内的任何合同不依赖于链接的库，则合同将被忽略。

```
// Deploy library LibA, then link LibA to contract B, then deploy B.
deployer.deploy(LibA);
deployer.link(LibA, B);
deployer.deploy(B);

// Link LibA to many contracts
deployer.link(LibA, [B, C, D]);
```



#### deployer.then

```
deployer.then(function() {...})
```

​	就像`promise`一样，运行任意部署步骤。 使用此选项可在迁移期间调用特定的合同函数，以添加，编辑和重新组织合同数据。

```
var a, b;
deployer.then(function() {
  // Create a new version of A
  return A.new();
}).then(function(instance) {
  a = instance;
  // Get the deployed instance of B
  return B.deployed();
}).then(function(instance) {
  b = instance;
  // Set the new instance of A's address on B via B's setA() function.
  return b.setA(a.address);
});
```



### 部署其他配置

#### solc

编译器配置：

```
compilers: {
  solc: {
    version: "0.5.1",
    settings: {
          optimizer: {
            enabled: true,
            runs: 200,
          }}}}
```



#### wallet

```
const HDWalletProvider = require('truffle-hdwallet-provider');
const fs = require('fs');
// 读取种子，12个单词组成的种子
const mnemonic = fs.readFileSync("./path/to/mnemonic.secret").toString().trim();

module.exports ={
    networks:{
        rinkebyTest:{
          provider: () => new HDWalletProvider(
            mnemonic, 
            `https://rinkeby.infura.io/v3/aa86f***60803c`,// you infura API key
            0, // 地址的起始索引
            10 // 生成的地址数量
          ),
          network_id: 4,
          // gas: 6500000,
          confirmations: 2,
          gasPrice: 5000000000, // 5 Gwei
          skipDryRun: true // 跳过预执行，直接部署
        }
    }
}
```