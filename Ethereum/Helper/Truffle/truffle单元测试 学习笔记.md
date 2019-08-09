## truffle 框架单元测试 学习笔记

### 零、环境

#### 1.依赖包

```
const { BN, constants, expectEvent, shouldFail } = require('openzeppelin-test-helpers');
```



#### 2.全局变量

```
const { ZERO_ADDRESS,MAX_UINT256,MAX_INT256,MIN_INT256} = constants;
```



#### 3.特殊的函数

```
// 每一个单元测试前(后)做的事情。
beforeEach(async function(){ ToDoSomeThing; });
afterEach(async function(){ ToDoSomeThing; });

// 总体的前面执行的函数，这个貌似没什么用
after(async function(){ ToDoSomeThing; });
before(async function(){ ToDoSomeThing; });
```



### 一、部署

#### 1.加载合约对象

```
const ContractName = artifacts.require('ContractName');
```



#### 2.部署新合约

```
this.contractName = await ContractName.new({from:accounts[0]});
```



#### 3.读取已部署合约

```
this.contractName = await ContractName.deployed();
```



### 二、合约交互

#### 1. 读取合约地址

```
// 不能合并成一句话，否则读取不到合约信息！！！
var contractObj = ContractName.deployed();
var contractAddr = contractObj.address;
```



#### 2. 函数交互

```
// 构建交易，默认使用accounts[0]作为签名者
it("测试交易级函数", async function(){
    var erc20 = ERC20.deployed();
    await erc20.transfer(userAddr, amountBN, {
      value: 0,
      from: accounts[0]
      });
    (await erc20.balanceOf(userAddr)).should.be.bignumber.equal(amountBN,'Error String');
});
```



#### 3. 捕获日志

```
it('测试捕获日志', async function(){
    var erc20 = ERC20.deployed();
  const { logs } = await erc20.transfer(userAddr, amountBN, {from: accounts[0]});
  expectEvent.inLogs(logs,'Transfer',{
      from: accounts[0],
      to: userAddr,
      value: amountBN
  });
});
```



#### 4.异常处理

错误类型：

```
reverting  :  revert
throwing   :  invalid opcode
outOfGas   :  out of gas
shouldFail :  3种中的一种
```

示例：

```
it('测试异常', async function(){
    var erc20 = ERC20.deployed();
    await shouldFail.reverting(erc20.transfer(to,amount),{from:accounts[1]});
    // 判断抛出异常的信息。
    await shouldFail.reverting.withMessage(erc20.function({ from: other }), "Unauthorized");
});
```



### 三、断言

#### 1.等于,不等于

```
it("test equal", async function(){
    var actualValue = await erc20.balanceOf(accounts[0]);
    var expected    = new BN("100");
    await actualValue.should.be.bignumber.equal(expected);
    //await actualValue.should.not.be.bignumber.equal(expected);
});
```



#### 2.比大小

```
actualValue.should.be.bignumber.above(expected);  // 实际 大于 期望， gt，greaterThan
above(2);   // 大于2
below(4);   // 小于4
at.least(3); // 最小为3
at.most(3);  // 
within(2,4);
```



#### 3.other

```
should.exist();
```



### 四、数值处理

#### 1.bignumber

```
// 新建，可以是数值，也可以是字符串，大值，只能用字符串。
var amount1 = new BN(100);
var amount2 = new BN("100000000000000000000000");
var amount3 = new BN('10').pow(new BN('22'));

// 运算,加普通数值要使用 addn,subn,muln,divn
amount1.add(amount2); // BN + BN 直接使用add
amount1.addn(100);

//断言判断时,需要加bignumber修饰，不然一定不相等。2个不同的BN实例
amount1.should.be.bignumber.equal(amount2);
```



#### 2.pow

```
MAX_UINT256 = new BN('2').pow(new BN('256')).sub(new BN('1'));
MAX_INT256  = new BN('2').pow(new BN('255)).sub(new BN('1'));
MIN_INT256  = new BN('2').pow(new BN('255)).mul(new BN('-1'));
```



#### 3.time

```
const { BN, expectEvent, shouldFail, time } = require('openzeppelin-test-helpers');
var _time = (await time.latest()).add(time.duration.weeks(1));

const duration = {
  seconds: function (val) { return new BN(val); },
  minutes: function (val) { return new BN(val).mul(this.seconds('60')); },
  hours: function (val) { return new BN(val).mul(this.minutes('60')); },
  days: function (val) { return new BN(val).mul(this.hours('24')); },
  weeks: function (val) { return new BN(val).mul(this.days('7')); },
  years: function (val) { return new BN(val).mul(this.days('365')); },
};

  before(async function () {
    // 在ganache解释的solid“now”函数中，前进到下一个块，正确读取时间
    await time.advanceBlock();
  });
```



##### time.advanceBlock()

强制挖一个块，增加块的高度。

##### time.latest()

Returns the timestamp of the latest mined block. Should be coupled with `advanceBlock` to retrieve the current blockchain time.

##### time.latestBlock()

.返回最新块的编号

##### time.increase(duration)

Increases the time of the blockchain by `duration` (in seconds), and mines a new block with that timestamp.

##### time.increaseTo(target)

Same as `increase`, but a target time is specified instead of a duration.

##### time.duration

Helpers to convert different time units to seconds. Available helpers are: `seconds`, `minutes`, `hours`, `days`, `weeks` and `years`.



#### 4.ether

```
const value = ether('42');
```