# solidity五种经典设计模式

设计模式是许多开发场景中的首选解决方案，本文将介绍五种经典的智能合约设计模式并给出 以太坊solidity实现代码：自毁合约、工厂合约、名称注册表、映射表迭代器和提款模式。



### 1、自毁合约

合约自毁模式用于终止一个合约，这意味着将从区块链上永久删除这个合约。 一旦被销毁，就不可能 调用合约的功能，也不会在账本中记录交易。

现在的问题是：“为什么我要销毁合约？”。

有很多原因，比如某些定时合约，或者那些一旦达到里程碑就必须终止的合约。 一个典型的案例 是贷款合约，它应当在贷款还清后自动销毁；另一个案例是基于时间的拍卖合约，它应当在拍卖结束后 终止 —— 假设我们不需要在链上保存拍卖的历史记录。

在处理一个被销毁的合约时，有一些需要注意的问题： 

合约销毁后，发送给该合约的交易将失败任何发送给被销毁合约的资金，都将永远丢失为避免资金损失，应当在发送资金前确保目标合约仍然存在，移除所有对已销毁合约的引用。 现在我们来看看代码：



```
contract SelfDesctructionContract {
  public address owner;
  public string someValue;
  modifier ownerRestricted {
     require(owner == msg.sender);
     _;
  } 
  
  // constructor
  function SelfDesctructionContract() {
     owner = msg.sender;
  }

  // a simple setter function
  function setSomeValue(string value){
     someValue = value;
  } 

  // you can call it anything you want
  function destroyContract() ownerRestricted {
    suicide(owner);
  }
}
```



正如你所看到的， destroyContract()方法负责销毁合约。

请注意，我们使用自定义的ownerRestricted修饰符来显示该方法的调用者，即仅允许合约的拥有者 销毁合约。



### 2、工厂合约

工厂合约用于创建和部署“子”合约。 这些子合约可以被称为“资产”，可以表示现实生活中的房子或汽车。

工厂用于存储子合约的地址，以便在必要时提取使用。 你可能会问，为什么不把它们存在Web应用数据库里？ 这是因为将这些地址数据存在工厂合约里，就意味着是存在区块链上，因此更加安全，而数据库的损坏 可能会造成资产地址的丢失，从而导致丢失对这些资产合约的引用。 除此之外，你还需要跟踪所有新 创建的子合约以便同步更新数据库。

工厂合约的一个常见用例是销售资产并跟踪这些资产（例如，谁是资产的所有者）。 需要向负责部署资产的 函数添加payable修饰符以便销售资产。 代码如下：



```
contract CarShop {
  address[] carAssets;
  function createChildContract(string brand, string model) public payable {
     // insert check if the sent ether is enough to cover the car asset ...
     address newCarAsset = new CarAsset(brand, model, msg.sender);            
     carAssets.push(newCarAsset);   
  }
  function getDeployedChildContracts() public view returns (address[]) {
     return carAssets;
  }
}

contract CarAsset {
  string public brand;
  string public model;
  address public owner;
  function CarAsset(string _brand, string _model, address _owner) public {
     brand = _brand;
     model = _model;
     owner = _owner;
  }
}
```



代码address newCarAsset = new CarAsset(...)将触发一个交易来部署子合约并返回该合约的地址。 由于工厂合约和资产合约之间唯一的联系是变量address[] carAssets，所以一定要正确保存子合约的地址。



### 3、名称注册表

假设你正在构建一个依赖与多个合约的DApp，例如一个基于区块链的在线商城，这个DApp使用了 ClothesFactoryContract、GamesFactoryContract、BooksFactoryContract等多个合约。

现在想象一下，将所有这些合约的地址写在你的应用代码中。 如果这些合约的地址随着时间的推移而变化，那该怎么办？

这就是名称注册表的作用，这个模式允许你只在代码中固定一个合约的地址，而不是数十、数百甚至数千个 地址。它的原理是使用一个合约名称 => 合约地址的映射表，因此可以通过调用getAddress("ClothesFactory") 从DApp内查找每个合约的地址。 使用名称注册表的好处是，即使更新那些合约，DApp也不会受到任何影响，因为 我们只需要修改映射表中合约的地址。代码如下：



```
contract NameRegistry {
  struct ContractDetails {
     address owner;
     address contractAddress;
     uint16 version;
  }
  mapping(string => ContractDetails) registry;
  function registerName(string name, address addr, uint16 ver) returns (bool) {
     // versions should start from 1
     require(ver >= 1);
     
     ContractDetails memory info = registry[name];
     require(info.owner == msg.sender);
     // create info if it doesn't exist in the registry
      if (info.contractAddress == address(0)) {
         info = ContractDetails({
            owner: msg.sender,
            contractAddress: addr,
            version: ver
         });
      } else {
         info.version = ver;
         info.contractAddress = addr;
      }
      // update record in the registry
      registry[name] = info;
      return true;
  }
   function getContractDetails(string name) constant returns(address, uint16) {
     return (registry[name].contractAddress, registry[name].version);
  }
}
```



你的DApp将使用getContractDetails(name)来获取指定合约的地址和版本。



### 4、映射表迭代器

很多时候我们需要对一个映射表进行迭代操作 ，但由于Solidity中的映射表只能存储值， 并不支持迭代，因此映射表迭代器模式非常有用。 需要指出的是，随着成员数量的增加， 迭代操作的复杂性会增加，存储成本也会增加，因此请尽可能地避免迭代。

实现代码如下：



```
contract MappingIterator {
  mapping(string => address) elements;
  string[] keys;
  function put(string key, address addr) returns (bool) {
     bool exists = elements[key] == address(0)
     if (!exists) {
        keys.push(key);
     }
     elements[key] = addr;
     return true;
   }
   function getKeyCount() constant returns (uint) {
      return keys.length;
   }
   function getElementAtIndex(uint index) returns (address) {
      return elements[keys[index]];
   }
   function getElement(string name) returns (address) {
      return elements[name];
   }
}
```



实现put()函数的一个常见错误，是通过遍历来检查指定的键是否存在。正确的做法是 elements[key] == address(0)。虽然遍历检查的做法不完全是一个错误，但它并不可取， 因为随着keys数组的增长，迭代成本越来越高，因此应该尽可能避免迭代。



### 5、提款模式

假设你销售汽车轮胎，不幸的是卖出的所有轮胎出问题了，于是你决定向所有的买家退款。

假设你跟踪记录了合约中的所有买家，并且合约有一个refund()函数，该函数会遍历所有买家 并将钱一一返还。

你可以选择 - 使用buyerAddress.transfer()或buyerAddress.send() 。 这两个函数的区别在于， 在交易异常时，send()不会抛出异常，而只是返回布尔值false ，而transfer()则会抛出异常。

为什么这一点很重要？

假设大多数买家是外部账户（即个人），但一些买家是其他合约（也许是商业）。 假设在 这些买方合约中，有一个合约，其开发者在其fallback函数中犯了一个错误，并且在被调用时抛出一个异常， fallback()函数是合约中的默认函数，如果将交易发送到合同但没有指定任何方法，将调用合约 的fallback()函数。 现在，只要我们在refund函数中调用contractWithError.transfer() ，就会抛出 异常并停止迭代遍历。 因此，任何一个买家合约的fallback()异常都将导致整个退款交易被回滚， 导致没有一个买家可以得到退款。

虽然在一次调用中退款所有买家可以使用send()来实现，但是更好的方式是提供withdrawFunds()方法，它 将单独按需要退款给调用者。 因此，错误的合约不会应用其他买家拿到退款。

实现代码如下：



```
contract WithdrawalContract {
   mapping(address => uint) buyers;
   function buy() payable {
      require(msg.value > 0);
      buyers[msg.sender] = msg.value;
   }
   function withdraw() {
      uint amount = buyers[msg.sender];
      require(amount > 0);
      buyers[msg.sender] = 0;      
      require(msg.sender.send(amount));
   }
}
```