[TOC]



## (1) account

```
create | c 
        创建一个账户. Returns reference ID to use in other operations
list | la 
        Print all accounts that were created or loaded
recover | r <file_path>
        Recover Libra wallet from the file path
write | w <file_path>
        Save Libra wallet mnemonic recovery seed to disk
mint | mintb | m | mb <receiver_account_ref_id>|<receiver_account_address> <number_of_coins>
        Mint coins to the account. Suffix 'b' is for blocking
```



### 1) create    创建地址

```
create | c  创建一个账户. Returns reference ID to use in other operations
```

创建帐户，使用CLI创建帐户不会更新区块链，只会创建本地密钥对。

```bash
libra% account create |  a c
>> Creating/retrieving next account from wallet
Created/retrieved account #0 address 4efbea83831455cf062178424768cf7edb4844374fc1521d48351374041bdd81
```

0是Alice帐户的索引，十六进制字符串是Alice帐户的地址。 索引只是引用Alice帐户的一种方式。 帐户索引是本地CLI索引，可以在其他CLI命令中使用，以便用户方便地引用他们创建的帐户。 该指数对区块链毫无意义。 只有当通过铸币（挖矿）将任何一笔钱添加到Alice的账户时，才会在区块链上创建Alice的账户，或者通过来自另一个用户的转账将钱转移到Alice的账户。 请注意，您也可以在CLI命令中使用十六进制地址。 帐户索引只是帐户地址的方便显示。

### 2) list          列举地址列表

要列出您创建的帐户.

```
libra% account list | a la
```

响应

```
User account index: 0, address: 4efbea83831455cf062178424768cf7edb4844374fc1521d48351374041bdd81, sequence number: 0, status: Local
User account index: 1, address: 79c7b144c3621cb7547acac4353a33365fdb3887543d19e9bc03f2f7d4c7aa06, sequence number: 0, status: Local
```

### 3) recover  恢复助记词

```
recover | r <file_path>
```

从文件路径中恢复Libra钱包

### 4) write      保存助记词

```
write | w <file_path>
```

将Libra钱包的助记符种子保存到磁盘

```
libra% account write /home/justin//mnt/seed
>> Saving Libra wallet mnemonic recovery seed to disk
Saved mnemonic seed to disk
```

助记词：

```
enlist rug weekend light refuse transfer uniform toilet gesture like truly rain lift duck glad primary top federal father badge antique south blind exotic;3
```

### 5) mint       挖矿

铸币。在testnet上创建和铸币是通过Faucet完成的。 Faucet是一种与testnet一起运行的服务。 此服务仅用于为testnet创建硬币，并不用运行在[主网]上. 

```
# 挖110个libra币给account[0], 仅限测试网
libra% account mint 0 110
```

响应

```
>> Minting coins
Mint request submitted
```





## (2) query

```
balance | b <account_ref_id>|<account_address>
        获取帐户的当前余额
        
sequence | s <account_ref_id>|<account_address> [reset_sequence_number=true|false]
        获取帐户的当前序列号，并在CLI中重置当前序列号（可选，默认为false）
        
account_state | as <account_ref_id>|<account_address>
        获取帐户的最新状态
        
txn_acc_seq | ts <account_ref_id>|<account_address> <sequence_number> <fetch_events=true|false>
        按帐户和序列号获取已提交的事务。 （可选）还可以获取此事务发出的事件。
        
txn_range | tr <start_version> <limit> <fetch_events=true|false>
        按版本范围获取已提交的事务。 （可选）还可以获取这些事务发出的事件
        
event | ev <account_ref_id>|<account_address> <sent|received> <start_sequence_number> <ascending=true|false> <limit>
        按帐户和事件类型获取事件（已发送|已接收）。

```

### 1) balance             查地址余额

获取帐户的当前余额

```
# 查询account[0]的余额
libra% query balance 0 | q b 0
```

响应

```
Balance is: 220.000000
```

### 2) sequence          查地址序列

```
sequence | s account [reset_sequence_number=true|false]   
```

获取帐户的当前序列号，并在CLI中重置当前序列号（可选，默认为false）,当助记词是导入时，序列号默认是0，需要通过此命令重新获取序列号。

```
# 查询account[0]的序列号
libra% query sequence 0
```

在 `query sequence 0`, 0是Alice的帐户的索引。 Alice和Bob的帐户的序列号都为0，即表示到目前为止尚未执行Alice或Bob的帐户中的任何交易。

### 3) account_state  查地址状态

```
account_state | as <account>
```

获取帐户的最新状态

```
# 查询一个新账户的状态
libra% query account_state 2
>> Getting latest account state
Latest account state is: 
 Account: 81d4efb00a58e0cc943c2a410e09cf793c9c9b5b663297d855f916dd96111bfd
 State: None
 Blockchain Version: 211447
```

查询一个已有交易的账户的状态

```
libra% q as 1
>> Getting latest account state
Latest account state is: 
 Account: 79c7b144c3621cb7547acac4353a33365fdb3887543d19e9bc03f2f7d4c7aa06
 State: Some(
    AccountStateBlob { 
     Raw: 0x010000002100000001217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97450000002000000079c7b144c3621cb7547acac4353a33365fdb3887543d19e9bc03f2f7d4c7aa06405973070000000000030000000000000000000000000000000000000000000000 
     Decoded: AccountResource {
        balance: 125000000,
        sequence_number: 0,
        authentication_key: 0x79c7b144c3621cb7547acac4353a33365fdb3887543d19e9bc03f2f7d4c7aa06,
        sent_events_count: 0,
        received_events_count: 3,
        delegated_withdrawal_capability: false,
    } 
     },
)
 Blockchain Version: 211483
```

### 4) txn_acc_seq     查交易详情

```
txn_acc_seq | ts <account> <sequence_number> <fetch_events=true|false>
```
查询交易,  按帐户和序列号获取已提交的事务。 （可选）还可以获取此事务发出的事件。

```
# 查询 account[0] 的第 1 笔交易 包括日志(true)
libra% query txn_acc_seq 0 0 true
```

### 5) txn_range

### 6) event                查日志详情

```
event | ev <account> <sent|received> <start_sequence_number> <ascending=true|false> <limit>
```

按帐户和事件类型获取事件（已发送|已接收）。

```
# 查询account[0]转账的日志，从第0开始，增序，查询10条
query event 0 sent 0 true 10
```



## (3) transfer

```
transfer | transferb | t | tb 
        <sender_account> <receiver_account> <number_of_coins> [gas_unit_price_in_micro_libras (default=0)] [max_gas_amount_in_micro_libras (default 100000)] Suffix 'b' is for blocking. 
```

转账操作：transfer不阻塞交易，transferb是阻塞等待交易返回状态。gas价格和gas数量默认。

```
# account[0]给account[1]转5个币
libra% transfer 0 1 5
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 0 <fetch_events=true|false>
```

 你可以使用 `transferb` 命令 (如下所示), 而不是 `transfer` 命令。只有将交易提交到区块链上 `transferb` 命令才会提交交易并给客户端反馈响应显示。

## (4) dev

```
Use the following args for this command:

compile | c <sender_account_address>|<sender_account_ref_id> <file_path> [is_module (default=false)] [output_file_path (compile into tmp file by default)]
        Compile move program

publish | p <sender_account_address>|<sender_account_ref_id> <compiled_module_path>
        Publish move module on-chain

execute | e <sender_account_address>|<sender_account_ref_id> <compiled_module_path> [parameters]
        Execute custom move script

```

### 1) compile

### 2) publish

### 3) execute