## Fabric Stub 接口文档
最后更新于2019/07，基于1.4

[TOC]

[官方API文档](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim)

### 一、参数相关接口

#### (1) GetFunctionAndParameters

从链码请求中获取参数，将第一个参数作为函数名返回，其余参数作为字符串数组中的参数返回。

```
GetFunctionAndParameters() (string, []string)
```

#### (2) GetArgs

从链码请求中获取参数，返回一个`[][]byte`类型,用于链码的Init和Invoke中。等价于getStringArgs()。

```
GetArgs() [][]byte
```

#### (3) GetStringArgs

从链码请求中获取参数，返回一个`[]string`类型,用于链码的Init和Invoke中。

```
GetStringArgs() []string
```

#### (4) GetArgsSlice

从链码请求中获取参数，返回一个`[]byte`类型,用于链码的Init和Invoke中。

```
GetArgsSlice() ([]byte, error)
```

### 二、交易信息解析接口

#### (1) GetChannelID

返回链码处理提议的通道ID，通常是交易提议中的channel_id字段值， 除非链码被另外一个通道调用。

```
GetChannelID() string
```

#### (2) GetCreator

返回链码调用提交者的身份对象，类型为ProposalCreator。 获取提交交易的身份信息（包括 MSP和证书信息）

```
GetCreator() ([]byte, error)
```

GetCreator returns `SignatureHeader.Creator` (e.g. an identity) of the `SignedProposal`. This is the identity of the agent (or user) submitting the transaction.

```
Org1MSP�-----BEGIN CERTIFICATE-----                             
MIICKjCCAdCgAwIBAgIQE6WvS6pil9SNjy5oi2TZIDAKBggqhkjOPQQDAjBzMQsw
CQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy
YW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu
b3JnMS5leGFtcGxlLmNvbTAeFw0xOTA3MDIwMzI4MDBaFw0yOTA2MjkwMzI4MDBa
MGwxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T
YW4gRnJhbmNpc2NvMQ8wDQYDVQQLEwZjbGllbnQxHzAdBgNVBAMMFkFkbWluQG9y
ZzEuZXhhbXBsZS5jb20wWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAARtCJZOdsDN
mA3BohG93qjcoQ0p/hm8le5jpDhsMK5FawlGo+lLTljSn6FO/V7mLdSVjotndAcz
1waJM+REDwsao00wSzAOBgNVHQ8BAf8EBAMCB4AwDAYDVR0TAQH/BAIwADArBgNV
HSMEJDAigCAL7+3jbUxg/efg8Tyd8eQssVGVcprHPhrsz8i7nRPynTAKBggqhkjO
PQQDAgNIADBFAiEAhbAH0P53VBBE6PV6rkyQrwSzsBg8D1C54WuP3aeRteMCICk8
uYQ4q3kh9AI2iRKExHqY/NmgftPonkPvl3YS0dvi                        
-----END CERTIFICATE-----                                       
```

#### (3) GetTxID

返回链码调用请求的交易ID（tx_id），交易ID可以在通道范围 内唯一标识一个交易。

```
GetTxID() string
```

返回值：

```
7d2599fba43e11a5d530b6fe98e928998ada3d3f84dbb5fb01db40aa6824d728
```

#### (4) GetTxTimestamp

返回创建交易时的时间戳。

```
GetTxTimestamp() (*timestamp.Timestamp, error)
```

这是从事务ChannelHeader中提取的，因此它将指示客户端的时间戳，并且在所有背书器中具有相同的值。

```
seconds:1562136262 nanos:588440974
```

#### (5) GetSignedProposal

方法返回完全解码的签名交易提议对象(SignedProposal)。该对象包含交易提案中的所有数据元素。

```
GetSignedProposal() (*pb.SignedProposal, error)
```

#### (6) GetBinding

获取交易的绑定信息，包含随机数和提交交易的身份信息等

```
GetBinding() ([]byte, error)
```

GetBinding returns the transaction binding, which is used to enforce a link between application data (like those stored in the transient field above) to the proposal itself. This is useful to avoid possible replay attacks.

GetBinding返回事务绑定，用于强制应用程序数据（如上面的瞬态字段中存储的数据）与提案本身之间的链接。 这有助于避免可能的重播攻击。

GetBinding返回事务绑定，事务绑定用于将应用程序数据(如上面存储在transient字段中的数据)与提案本身之间的链接强制执行。这对于避免可能的重播攻击很有用。

#### (7) GetDecorations

getdecoration返回关于来自节点的提案的附加数据(如果适用)。这个数据是由节点的装饰器设置的，它附加或修改传递给chain代码的chain代码输入。

```
GetDecorations() map[string][]byte
```

GetDecorations returns additional data (if applicable) about the proposal that originated from the peer. This data is set by the decorators of the peer, which append or mutate the chaincode input passed to the chaincode.

GetDecorations返回有关源自节点的提案的其他数据（如果适用）。 此数据由节点的装饰器设置，它附加或改变传递给链代码的链代码输入。

getdecoration返回关于来自节点的提案的附加数据(如果适用)。这个数据是由节点的装饰器设置的，它附加或修改传递给chain代码的chain代码输入。

### 三、普通键状态操作

#### (1) PutState

将指定的`key`和`value`放入数据-写入型的交易议案的`writeset`中。

```
PutState(key string, value []byte) error
```

在交易验证并成功提交之前，PutState不会影响账本。 简单键不能是空字符串，并且不能以空字符（0x00）开头，以避免范围查询与复合键冲突，复合键在内部以0x00为前缀作为复合键名称空间。

#### (2) GetState

从账本中返回指定`key`的当前值。

```
GetState(key string) ([]byte, error)
```

请注意，GetState不会从writeset读取数据，而writeset尚未提交到分类帐。 换句话说，GetState不会考虑由PutState修改的尚未提交的数据。 如果状态数据库中不存在该键，则返回（nil，nil）。

#### (3) DelState

DelState记录要在交易提案的writeset中删除的指定“键”。

```
DelState(key string) error
```

当事务验证并成功提交时，`key`及其值将从分类帐中删除。

#### (4) GetHistoryForKey

查询一个建的历史数据。返回一个`historyQueryIterator`类型的迭代器。

```
GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error)
```

接口应该只用于只读操作中。不应该把这个接口的返回数据构建交易，因为查询过程中，可以由于其他交易更新了键的值，影响了结果集，而查询只会获取已提交的数据，可能造成数据不准确。

HistoryQueryIteratorInterface 可获取的数据

- TxId  交易hash
- Timestamp  交易的时间戳，交易的提案头里面，由客户端填入的。  
- Value  值，这笔交易执行后，这个key的值。
- IsDelete 布尔值，删除标志位

```go
func getDataFromHistory(results shim.HistoryQueryIteratorInterface) ([]HistoryItem, error) {
	var itemList []HistoryItem

	for results.HasNext() {
		itemData, err := results.Next()
		if err != nil {
			return nil, err
		}

		var item HistoryItem
		item.Txid = itemData.TxId

		var order Order
		err = json.Unmarshal(itemData.Value, &order)
		if err != nil {
			return nil, err
		}
		item.Order = order
		item.Hash = order.Order_hash

		txTimestamp := itemData.Timestamp
		item.Timestamp = time.Unix(txTimestamp.Seconds, int64(txTimestamp.Nanos)).Format(TimeFMT)

		itemList = append(itemList, item)
	}
	return itemList, nil
}
```

#### (5) GetStateByRange

查询状态数据库里键在［startKey, endKey) 之间的值，包含 tartKey ，不包含 end Key 

请注意，startKey和endKey可以是空字符串，这意味着在开始或结束时无限范围查询。

```
GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)
```

GetStateByRange返回分类帐中一组键的范围迭代器。 迭代器可用于迭代startKey（包含）和endKey（独占）之间的所有键。 但是，如果startKey和endKey之间的键数大于totalQueryLimit（在core.yaml中定义），则此迭代器不能用于获取所有键（结果将由totalQueryLimit限制）。 迭代器以词法顺序返回键。  完成后，在返回的StateQueryIteratorInterface对象上调用Close（）。 在验证阶段重新执行查询，以确保自事务认可（检测到幻像读取）后结果集未发生更改。

#### (6) GetStateByRangeWithPagination

```
GetStateByRangeWithPagination(startKey, endKey string, pageSize int32, bookmark string) (StateQueryIteratorInterface, *pb.QueryResponseMetadata, error)
```

GetStateByRangeWithPagination返回分类帐中一组键的范围迭代器。迭代器可用于获取startKey（包含）和endKey（不包括）之间的密钥。当空字符串作为值传递给bookmark参数时，返回的迭代器可用于获取startKey（包含）和endKey（不包括）之间的第一个`pageSize`键。当书签是非空的字符串时，迭代器可用于获取书签（包括）和endKey（不包括）之间的第一个`pageSize`键。请注意，只有查询结果的先前页面中存在的书签（ResponseMetadata）可以用作书签参数的值。否则，必须将空字符串作为书签传递。迭代器以词法顺序返回键。请注意，startKey和endKey可以是空字符串，这意味着在开始或结束时无限范围查询。完成后，在返回的StateQueryIteratorInterface对象上调用Close（）。此调用仅在只读事务中受支持。

### 四、复合键接口

复合建用于联合查询

#### (1) CreateCompositeKey

构造组合键，将给定的“属性”组合起来形成一个复合键。

- objectType  复合键前缀，string
- attributes   要拼接到复合键的各属性值，string数组

```
CreateCompositeKey(objectType string, attributes []string) (string, error)
```

objectType和attributes应该都是有效的utf8字符串，并且不应该包含U+0000 (nil字节)和U+10FFFF(最大且未分配的代码点)。生成的复合键可以用作PutState()中的键。

```go
// 定义组合键名
const IndexName = "color~id"

// 创建组合键
colorNameIndexKey, err := stub.CreateCompositeKey(IndexName, []string{car.Color, car.ID})
if err != nil {
	return shim.Error("Fail to create Composite key")
}

// 将索引信息保保存在Key中
err = stub.PutState(colorNameIndexKey, []byte{0x00})
if err != nil {
	return shim.Error("Fail to put Composite key")
}
```

#### (2) SplitCompositeKey

拆分复合建，建其还原成组合时的属性。

```
SplitCompositeKey(compositeKey string) (string, []string, error)
```

Composite keys found during range queries or partial composite key queries can therefore be split into their composite parts.

 因此，在范围查询或部分复合键查询期间找到的复合键可以拆分为它们的复合部分。

```
colorIdKey, err := StateQueryIteratorInterface.Next()
if err != nil {
	return shim.Error(err.Error())
}
objectType, compisiteKeys, err := stub.SplitCompositeKey(string(colorIdKey.Key))
returnColor := compisiteKeys[0]
returnId := compisiteKeys[1]	
```

#### (3) GetStateByPartialCompositeKey

部分组合键查询，通过组合键中的一部分成员查询组合键。

```
GetStateByPartialCompositeKey(objectType string, keys []string) (StateQueryIteratorInterface, error)
```

GetStateByPartialCompositeKey根据给定的部分复合键查询分类帐中的状态。 此函数返回一个迭代器，该迭代器可用于迭代其前缀与给定的部分复合键匹配的所有复合键。 但是，如果匹配的复合键的数量大于totalQueryLimit（在core.yaml中定义），则此迭代器不能用于获取所有匹配的键（结果将受totalQueryLimit的限制）。 期望`objectType`和属性只有有效的utf8字符串，不应包含U + 0000（零字节）和U + 10FFFF（最大和未分配的代码点）。 请参阅相关函数SplitCompositeKey和CreateCompositeKey。 完成后，在返回的StateQueryIteratorInterface对象上调用Close（）。 在验证阶段重新执行查询，以确保自事务认可（检测到幻像读取）后结果集未发生更改。

```go
// const IndexName = "color~id"
color := args[0]

//返回包含给出颜色的组合键的迭代器
StateQueryIteratorInterface, err := stub.GetStateByPartialCompositeKey(IndexName, []string{color})
if err != nil {
	return shim.Error(err.Error())
}
```

#### (4) GetStateByPartialCompositeKeyWithPagination

通过分页的部分组合键获取状态

```
GetStateByPartialCompositeKeyWithPagination(objectType string, keys []string, pageSize int32, bookmark string) (StateQueryIteratorInterface, *pb.QueryResponseMetadata, error)
```

GetStateByPartialCompositeKeyWithPagination基于给定的部分复合键查询分类帐中的状态。此函数返回一个迭代器，该迭代器可用于迭代前缀与给定的部分复合键匹配的复合键。当空字符串作为值传递给bookmark参数时，返回的迭代器可用于获取其前缀与给定的部分复合键匹配的第一个`pageSize`复合键。当书签是非空的字符串时，迭代器可用于获取书签（包括）和最后匹配的复合键之间的第一个`pageSize`键。请注意，只有查询结果的先前页面中存在的书签（ResponseMetadata）可以用作书签参数的值。否则，必须将空字符串作为书签传递。期望`objectType`和属性只有有效的utf8字符串，不应包含U + 0000（零字节）和U + 10FFFF（最大和未分配的代码点）。请参阅相关函数SplitCompositeKey和CreateCompositeKey。完成后，在返回的StateQueryIteratorInterface对象上调用Close（）。此调用仅在只读事务中受支持。

### 五、Transient 接口

#### (1) GetTransient

获取一些私密信息，这部分信息不会写入账本数据，返回可以被链码使用但没有保存在账本中的 临时映射表，例如用于加密和解密的密码学信息。

```
GetTransient() (map[string][]byte, error)
```

GetTransient返回`ChaincodeProposalPayload.Transient`字段。 它是包含可用于实现某种形式的应用程序级机密性的数据（例如加密材料）的映射。 根据`ChaincodeProposalPayload`的规定，该字段的内容应始终从事务中省略，并从账本中排除。

#### (2) PutPrivateData

PutPrivateData将指定的`key`和`value`放入交易的私有writeset中。

```
PutPrivateData(collection string, key string, value []byte) error
```

 请注意，只有私有写入集的哈希进入事务提议响应（发送给发出事务的客户端），并且实际的私有写入集临时存储在临时存储中。 在事务验证并成功提交之前，PutPrivateData不会影响`collection`。

#### (3) GetPrivateData

从指定的集合(collection)中返回指定状态键(key)的值。

```
GetPrivateData(collection, key string) ([]byte, error)
```

请注意，GetPrivateData不会从私有writeset读取数据，而私有writeset尚未提交到`collection`。 换句话说，GetPrivateData不会考虑PutPrivateData修改的尚未提交的数据。

#### (4) DelPrivateData

在交易私有写入集中记录指定键被删除。

```
DelPrivateData(collection, key string) error
```

请注意，只有私有写入集的哈希进入事务提议响应（发送给发出事务的客户端），并且实际的私有写入集临时存储在临时存储中。 在验证并成功提交事务时，将从集合中删除`key`及其值。

#### (5) GetPrivateDataByRange

```
GetPrivateDataByRange(collection, startKey, endKey string) (StateQueryIteratorInterface, error)
```

GetPrivateDataByRange returns a range iterator over a set of keys in a given private collection. The iterator can be used to iterate over all keys between the startKey (inclusive) and endKey (exclusive). The keys are returned by the iterator in lexical order. Note that startKey and endKey can be empty string, which implies unbounded range query on start or end. Call Close() on the returned StateQueryIteratorInterface object when done. The query is re-executed during validation phase to ensure result set has not changed since transaction endorsement (phantom reads detected).

GetPrivateDataByRange返回给定私有集合中的一组键的范围迭代器。 迭代器可用于迭代startKey（包含）和endKey（独占）之间的所有键。 迭代器以词法顺序返回键。 请注意，startKey和endKey可以是空字符串，这意味着在开始或结束时无限范围查询。 完成后，在返回的StateQueryIteratorInterface对象上调用Close（）。 在验证阶段重新执行查询，以确保自事务认可（检测到幻像读取）后结果集未发生更改。

GetPrivateDataByRange在给定私有集合中的一组键上返回一个范围迭代器。迭代器可用于迭代startKey(包含)和endKey(排除)之间的所有键。迭代器按词法顺序返回键。注意，startKey和endKey可以是空字符串，这意味着开始或结束时的范围查询是无界的。完成后，对返回的StateQueryIteratorInterface对象调用Close()。在验证阶段重新执行查询，以确保自事务背书(检测到幻像读取)以来结果集没有更改。

#### (7) GetPrivateDataByPartialCompositeKey

```
GetPrivateDataByPartialCompositeKey(collection, objectType string, keys []string) (StateQueryIteratorInterface, error)
```

GetPrivateDataByPartialCompositeKey queries the state in a given private collection based on a given partial composite key. This function returns an iterator which can be used to iterate over all composite keys whose prefix matches the given partial composite key. The `objectType` and attributes are expected to have only valid utf8 strings and should not contain U+0000 (nil byte) and U+10FFFF (biggest and unallocated code point). See related functions SplitCompositeKey and CreateCompositeKey. Call Close() on the returned StateQueryIteratorInterface object when done. The query is re-executed during validation phase to ensure result set has not changed since transaction endorsement (phantom reads detected).

GetPrivateDataByPartialCompositeKey基于给定的部分组合键查询给定私有集合中的状态。 此函数返回一个迭代器，该迭代器可用于迭代其前缀与给定的部分复合键匹配的所有复合键。 期望`objectType`和属性只有有效的utf8字符串，不应包含U + 0000（零字节）和U + 10FFFF（最大和未分配的代码点）。 请参阅相关函数SplitCompositeKey和CreateCompositeKey。 完成后，在返回的StateQueryIteratorInterface对象上调用Close（）。 在验证阶段重新执行查询，以确保自事务认可（检测到幻像读取）后结果集未发生更改。

GetPrivateDataByPartialCompositeKey根据给定的部分组合键查询给定私有集合中的状态。此函数返回一个迭代器，可用于迭代前缀与给定的部分组合键匹配的所有组合键。“objectType”和属性应该只有有效的utf8字符串，不应该包含U+0000 (nil字节)和U+10FFFF(最大且未分配的代码点)。参见相关函数SplitCompositeKey和CreateCompositeKey。完成后，对返回的StateQueryIteratorInterface对象调用Close()。在验证阶段重新执行查询，以确保自事务背书(检测到幻像读取)以来结果集没有更改。

### 六、CouchDB富查询

#### (1) GetQueryResult

根据指定条件查询状态数据库里存储的值，目前只有 CouchDB 才能支持

```
GetQueryResult(query string) (StateQueryIteratorInterface, error)
```

 GetQueryResult performs a "rich" query against a state database. It is only supported for state databases that support rich query, e.g.CouchDB. The query string is in the native syntax of the underlying state database. An iterator is returned which can be used to iterate over all keys in the query result set. However, if the number of keys in the query result set is greater than the totalQueryLimit (defined in core.yaml), this iterator cannot be used to fetch all keys in the query result set (results will be limited by the totalQueryLimit). The query is NOT re-executed during validation phase, phantom reads are not detected. That is, other committed transactions may have added, updated, or removed keys that impact the result set, and this would not be detected at validation/commit time.  Applications susceptible to this should therefore not use GetQueryResult as part of transactions that update ledger, and should limit use to read-only chaincode operations.

GetQueryResult对状态数据库执行“富”查询。它只支持支持富查询的状态数据库，例如couchdb。查询字符串使用底层状态数据库的本地语法。返回一个迭代器可以用来遍历查询结果集的所有钥匙。然而,如果查询结果集的键的数量大于totalQueryLimit core.yaml(定义),这个迭代器不能用于获取查询结果集的所有钥匙(totalQueryLimit)结果将是有限的。在验证阶段不会重新执行查询，也不会检测到幻像读取。也就是说，其他提交的事务可能添加、更新或删除了影响结果集的键，而这在验证/提交时不会检测到。因此，易受此影响的应用程序不应将GetQueryResult作为更新分类帐的事务的一部分使用，而应将使用限制为只读链代码操作。

GetQueryResult对状态数据库执行“丰富”查询。 它仅支持支持富查询的状态数据库，例如ContentDB。 查询字符串采用基础状态数据库的本机语法。 返回一个迭代器，可用于迭代查询结果集中的所有键。 但是，如果查询结果集中的键数大于totalQueryLimit（在core.yaml中定义），则此迭代器不能用于获取查询结果集中的所有键（结果将受totalQueryLimit限制）。 在验证阶段不重新执行查询，未检测到幻像读取。 也就是说，其他已提交的事务可能已添加，更新或删除了影响结果集的密钥，并且在验证/提交时不会检测到这一点。 因此，易受此影响的应用程序不应将GetQueryResult用作更新分类帐的事务的一部分，并应限制对只读链代码操作的使用。

#### (2) GetQueryResultWithPagination

```
GetQueryResultWithPagination(query string, pageSize int32, bookmark string) (StateQueryIteratorInterface, *pb.QueryResponseMetadata, error)
```

GetQueryResultWithPagination performs a "rich" query against a state database. It is only supported for state databases that support rich query, e.g., CouchDB. The query string is in the native syntax of the underlying state database. An iterator is returned which can be used to iterate over keys in the query result set. When an empty string is passed as a value to the bookmark argument, the returned iterator can be used to fetch the first `pageSize` of query results. When the bookmark is a non-emptry string, the iterator can be used to fetch the first `pageSize` keys between the bookmark and the last key in the query result. Note that only the bookmark present in a prior page of query results (ResponseMetadata) can be used as a value to the bookmark argument. Otherwise, an empty string must be passed as bookmark.This call is only supported in a read only 

GetQueryResultWithPagination对状态数据库执行“富”查询。它只支持支持富查询的状态数据库，例如CouchDB。查询字符串使用底层状态数据库的本地语法。返回一个迭代器，可用于迭代查询结果集中的键。当一个空字符串作为值传递给bookmark参数时，返回的迭代器可用于获取查询结果的第一个页面大小。当书签是非空字符串时，可以使用迭代器获取书签和查询结果中的最后一个键之间的第一个pageSize键。注意，只有查询结果的前一页中的书签(ResponseMetadata)可以用作bookmark参数的值。否则，必须将空字符串作为书签传递。此调用仅在只读中受支持

GetQueryResultWithPagination对状态数据库执行“丰富”查询。 它仅支持支持富查询的状态数据库，例如CouchDB。 查询字符串采用基础状态数据库的本机语法。 返回一个迭代器，可用于迭代查询结果集中的键。 当空字符串作为值传递给bookmark参数时，返回的迭代器可用于获取查询结果的第一个`pageSize`。 当书签是非空的字符串时，迭代器可用于获取书签和查询结果中的最后一个键之间的第一个`pageSize`键。 请注意，只有查询结果的先前页面中存在的书签（ResponseMetadata）可以用作书签参数的值。 否则，必须将空字符串作为书签传递。此调用仅在只读事务中受支持。

#### (3) GetPrivateDataQueryResult

```
GetPrivateDataQueryResult(collection, query string) (StateQueryIteratorInterface, error)
```

GetPrivateDataQueryResult performs a "rich" query against a given private collection. It is only supported for state databases that support rich query, e.g.CouchDB. The query string is in the native syntax of the underlying state database. An iterator is returned which can be used to iterate (next) over the query result set. The query is NOT re-executed during validation phase, phantom reads are not detected. That is, other committed transactions may have added, updated, or removed keys that impact the result set, and this would not be detected at validation/commit time.  Applications susceptible to this should therefore not use GetQueryResult as part of transactions that update ledger, and should limit use to read-only chaincode operations.

GetPrivateDataQueryResult对给定的私有集合执行“丰富”查询。 它仅支持支持富查询的状态数据库，例如ContentDB。 查询字符串采用基础状态数据库的本机语法。 返回一个迭代器，可用于在查询结果集上迭代（下一步）。 在验证阶段不重新执行查询，未检测到幻像读取。 也就是说，其他已提交的事务可能已添加，更新或删除了影响结果集的密钥，并且在验证/提交时不会检测到这一点。 因此，易受此影响的应用程序不应将GetQueryResult用作更新分类帐的事务的一部分，并应限制对只读链代码操作的使用。

GetPrivateDataQueryResult对给定的私有集合执行“富”查询。它只支持支持富查询的状态数据库，例如couchdb。查询字符串使用底层状态数据库的本地语法。返回一个迭代器，该迭代器可用于遍历查询结果集(next)。在验证阶段不重新执行查询，不检测幻像读取。也就是说，其他提交的事务可能添加、更新或删除了影响结果集的键，而这在验证/提交时不会检测到。因此，易受此影响的应用程序不应将GetQueryResult作为更新分类帐的事务的一部分使用，而应将使用限制为只读链代码操作。

### 七、配置功能接口

#### (1) SetEvent

设置事件的名称和内容

[事件框架](https://blog.csdn.net/maixia24/article/details/79699096)

```
SetEvent(name string, payload []byte) error
```

SetEvent allows the chaincode to set an event on the response to the proposal to be included as part of a transaction. The event will be available within the transaction in the committed block regardless of the validity of the transaction.

SetEvent允许链代码将对提案的响应设置为事件的一部分作为事务的一部分。 无论事务的有效性如何，事件都将在已提交块中的事务中可用。

SetEvent允许chain代码在响应建议时设置一个事件，该建议将被包含在事务中。无论事务的有效性如何，事件都将在提交的块中的事务中可用。

```go
// 一个设置合约日志的方法
var log *shim.ChaincodeLogger
func getLogger(stub shim.ChaincodeStubInterface) (c *shim.ChaincodeLogger) {
	fcn, _ := stub.GetFunctionAndParameters()
	c = shim.NewLogger(fmt.Sprintf("%s.%s.%s", stub.GetChannelID(), "token", fcn))
	c.SetLevel(shim.LogDebug)
	return
}
// 2019-07-03 06:32:35.835 UTC [mychannel.getstate] Infof -> INFO 004 args: [justin3]
```

#### (2) InvokeChaincode

```
InvokeChaincode(chaincodeName string, args [][]byte, channel string) pb.Response
```

InvokeChaincode使用相同的事务上下文在本地调用指定的链代码`Invoke`; 也就是说，chaincode调用chaincode不会创建新的事务消息。 如果被调用的链代码在同一个通道上，它只是将被调用的链代码读取集和写入集添加到调用事务中。 如果被调用的链代码在不同的通道上，则仅将响应返回给调用的链代码; 来自被调用链码的任何PutState调用都不会对分类账产生任何影响; 也就是说，不同通道上的被调用链代码不会将其读取集和写入集应用于事务。 只有调用链代码的读取集和写入集才会应用于事务。 有效地，在不同通道上调用的链代码是`Query`，它在后续提交阶段不参与状态验证检查。如果`channel`为空，则假定调用者的通道。

```go
// 把 string 的参数转成 [][]byte
func toChaincodeArgs(args ...string) [][]byte {
	bargs := make([][]byte, len(args))
	for i, arg := range args {
		bargs[i] = []byte(arg)
	}
	return bargs
}
```

### 八、其他

#### (1) SetStateValidationParameter

```
SetStateValidationParameter(key string, ep []byte) error
```

SetStateValidationParameter为“key”设置键级认可策略。

SetStateValidationParameter sets the key-level endorsement policy for `key`.

#### (2) GetStateValidationParameter

```
GetStateValidationParameter(key string) ([]byte, error)
```

GetStateValidationParameter检索“key”的键级认可策略。注意，这将在事务的readset中引入对“key”的读取依赖。

#### (3) SetPrivateDataValidationParameter

```
SetPrivateDataValidationParameter(collection, key string, ep []byte) error
```

SetPrivateDataValidationParameter sets the key-level endorsement policy for the private data specified by `key`.

SetPrivateDataValidationParameter为`key`指定的私有数据设置密钥级认可策略。

SetPrivateDataValidationParameter为“key”指定的私有数据设置键级认可策略。

#### (4) GetPrivateDataValidationParameter

```
GetPrivateDataValidationParameter(collection, key string) ([]byte, error)
```

GetPrivateDataValidationParameter retrieves the key-level endorsement policy for the private data specified by `key`. Note that this introduces a read dependency on `key` in the transaction's readset.

GetPrivateDataValidationParameter检索由`key`指定的私有数据的密钥级认可策略。 请注意，这会在事务的readset中引入对“key”的读取依赖性。

GetPrivateDataValidationParameter检索“key”指定的私有数据的键级认可策略。注意，这在事务的readset中引入了对“key”的读取依赖。









