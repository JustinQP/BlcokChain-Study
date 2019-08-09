来自[以太坊ERC20 Token标准完整说明](https://blog.csdn.net/sinat_34070003/article/details/79105284)

[TOC]

## 什么是ERC20 token?

市面上出现了大量的用ETH做的代币，他们都遵守REC20协议，那么我们需要知道什么是REC20协议。



### 概述

token代表数字资产，具有价值，但是并不是都符合特定的规范。

​	基于ERC20的货币更容易互换，并且能够在Dapps上相同的工作。

​	新的标准可以让token更兼容，允许其他功能，包括投票标记化。操作更像一个投票操作

​	Token的持有人可以完全控制资产，遵守ERC20的token可以跟踪任何人在任何时间拥有多少token.

​	基于eth合约的子货币，所以容易实施。只能自己去转让。





## ERC20 Token标准(Github)

### 序言

EIP: 20 
Title: ERC-20 Token Standard 
Author: Fabian Vogelsteller fabian@ethereum.org, Vitalik Buterin vitalik.buterin@ethereum.org 
Type: Standard 
Category: ERC 
Status: Accepted 
Created: 2015-11-19



### 总结

token的接口标准



### 抽象

以下标准允许在智能合约中实施标记的标记API。
该标准提供了转移token的基本功能，并允许token被批准，以便他们可以由另一个在线第三方使用。



### 动机

标准接口可以让Ethereum上的任何令牌被其他应用程序重新使用：从钱包到分散式交换。



### 规则

Token



### Function:

注意：调用者必须处理返回false的returns (bool success).
调用者绝对不能假设返回false的情况不存在。



#### 1.  name

返回这个令牌的名字，比如”MyToken”.

**可选** - 这种方法可以用来提高可用性，但接口和其他契约不能指望这些值存在。

```
function name() constant returns (string name)
```



#### 2. symbol

返回令牌的符号，比如HIX.

**可选** - 这种方法可以用来提高可用性，但接口和其他契约不能指望这些值存在。

```
function symbol() constant returns (string symbol)
```



#### 3. decimals

返回token使用的小数点后几位， 比如 8,表示分配token数量为100000000

**可选** - 这种方法可以用来提高可用性，但接口和其他契约不能指望这些值存在。

```
function decimals() constant returns (uint8 decimals)
```



#### 4. totalSupply

返回token的总供应量。**必选**

```
function totalSupply() constant returns (uint256 totalSupply)
```



#### 5. balanceOf

返回地址是_owner的账户的账户余额。**必选**

```
function balanceOf(address _owner) constant returns (uint256 balance)
```



#### 6. transfer

转移value的token数量到的地址to，并且必须触发Transfer事件。**必选**

如果_from帐户余额没有足够的令牌来支出，该函数应该被throw。_

创建新令牌的令牌合同应该在创建令牌时将`_from`地址设置为0x0触发传输事件。

```
function transfer(address *to, uint256* value) returns (bool success)
```

**注意:**  0值的传输必须被视为正常传输并触发交易事件。



#### 7. transferFrom

从地址from发送数量为value的token到地址_to,必须触发Transfer事件。**必选**

transferFrom方法用于提取工作流，允许合约代您转移token。

这可以用于例如允许合约代您转让代币和/或以子货币收取费用。

除了_from帐户已经通过某种机制故意地授权消息的发送者之外，该函数**应该**throw。

```
 function transferFrom(address from, address to, uint256 _value) returns (bool success)
```

**注意: ** 0值的传输必须被视为正常传输并触发传输事件。



#### 8. approve

允许spender多次取回您的帐户，最高达value金额。**必选**
如果再次调用此函数，它将以_value覆盖当前的余量。

```
function approve(address spender, uint256 value) returns (bool success)
```

**注意：**为了阻止向量攻击，客户端需要确认以这样的方式创建用户接口，即将它们设置为0，然后将其设置为同一个花费者的另一个值。



#### 9. allowance

返回spender仍然被允许从owner提取的金额。**必选**

```
function allowance(address owner, address spender) constant returns (uint256 remaining)
```



### Events:

#### 10. Transfer

当token被转移(包括0值)，必须被触发。**必选**

```
event Transfer(address indexed from, address indexed to, uint256 _value)
```



#### 11. Approval

当任何成功调用approve(address *spender, uint256* value)后，必须被触发。**必选**

```
event Approval(address indexed owner, address indexed spender, uint256 _value)
```





实施
在Ethereum网络上部署了大量符合ERC20标准的令牌。
具有不同权衡的各种团队已经编写了不同的实施方案：从节省gas到提高安全性。



## ERC20 标准接口示例    

以下是一个接口合同，声明所需的功能和事件以符合ERC20标准：

```
// https://github.com/ethereum/EIPs/issues/20 
contract ERC20 { 
    function totalSupply() public view returns (uint totalSupply); 
    function balanceOf(address _owner) public returns (uint balance); 
    function transfer(address to, uint value) public returns (bool success); 
    function transferFrom(address from, address to, uint _value) public returns (bool success); 
    function approve(address spender, uint value) public returns (bool success); 
    function allowance(address owner, address spender) view returns (uint remaining); 
    event Transfer(address indexed from, address indexed to, uint _value); 
    event Approval(address indexed owner, address indexed spender, uint _value); 
}
```