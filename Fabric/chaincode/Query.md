[TOC]



## 一、简单查询

### (1) 简单K-V存储

直接使用`stub.PutState(key, value)`方法写入数据。

一般情况下，在主键的key前面增加一个前缀，避免主键的内容和其他K-V存在冲突。

```go
const ID_Prefix = "Car_"
func (t *APICC) tPutState(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 4 {
		return shim.Error("Wrong number of parameters.")
	}
    
    id := args[0]
	key := ID_Prefix + id
	
	var car = &Car{
		ID:         args[0],
		Color:      args[1],
		Price:      args[2],
		LaunchDate: args[3],
	}
	
	value, err := json.Marshal(car)
	if err != nil {
		return shim.Error(err.Error())
	}
	
	err = stub.PutState(key, value)
	if err != nil {
		return shim.Error(err.Error())
	}
	
	return shim.Success(SUCCESS)
}
```

### (2) 简单K-V查询

再通过`stub.GetState(key)`方法读取数据。

```go
func (t *APICC) tGetPutState(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 1 {
		return shim.Error(ErrParamNo.Error())
	}

	key := ID_Prefix + args[0]
	dbValue, err := stub.GetState(key)
	if err != nil {
		return shim.Error(err.Error())
	}
	if dbValue == nil {
		return shim.Error("get car info error: value is null")
	}

	var car Car
	err = json.Unmarshal(dbValue, &car)
	if err != nil {
		return shim.Error("get car info error: unmarshal failed")
	}

	retBytes, err := json.Marshal(car)
	if err != nil {
		return shim.Error("get car info error: marshal failed")
	}

	return shim.Success(retBytes)
}
```



## 二、组合查询

在生产环境下，一般只有简单的查询，可能满足不了复杂的需要。因此引入复合建，建立索引。方便复合查询。

本示例为**leveldb**模式下使用。使用的合约数据结构：

```go
type Car struct {
	ID         string `json:"ID"`
	Color      string `json:"Color"`
	Price      string `json:"Price"`
	LaunchDate string `json:"LaunchDate"`
}
```

### (1)  简单K-V存储

直接使用`stub.PutState(key, value)`方法写入数据。同上。

### (2) 建立索引

通过创建组合键创建索引。在第一步的基础上，增加添加索引的一段逻辑

```go
const IndexName = "color~id"
//创建Color与ID的组合键  car 是第一步中存数Ledger的Car的实例
colorNameIndexKey, err := stub.CreateCompositeKey(IndexName, []string{car.Color, car.ID}) 
if err != nil {
	return shim.Error("Fail to create Composite key")
}

// 将索引信息保保存在Key中
stub.PutState(colorNameIndexKey, []byte{0x00}) 
```

### (3) 查找结果集

获取`GetStateByPartialCompositeKey`获取组合键的查询结果

```go
color := args[0]
colorIdResultsIterator, err := stub.GetStateByPartialCompositeKey(IndexName, []string{color})
```

### (4) 解析组合键

```go
// 逐个解析
colorIdKey, err := colorIdResultsIterator.Next()

//通过SplitCompositeKey 解析出Car的主键 ID
//  objectType  string 组合键的前缀（表名）
//  compisiteKeys []string 组合键的组成成员列表 [Color, ID]
objectType, compisiteKeys, err := stub.SplitCompositeKey(string(colorIdKey.Key))

// 获取主键 Car.ID 的值，再通过id查询对应的数据
ItemID := compisiteKeys[1]
```

### (5) 简单K-V查询

```go
var carItem Car
carBytes, err := stub.GetState(key)
err = json.Unmarshal(carBytes, &carItem)
```

### (6) 构建返回值

```go
var returnList []Car
for colorIdResultsIterator.HasNext() {
	// 解析和查询数据
	returnList = append(returnList, carItem)
}
retJson, err := json.Marshal(returnList)
if err != nil {
	return shim.Error(err.Error())
}
return shim.Success(retJson)
```

### (7) 示例

```go
type Car struct {
	ID         string `json:"ID"`
	Color      string `json:"Color"`
	Price      string `json:"Price"`
	LaunchDate string `json:"LaunchDate"`
}

const ID_Prefix = "Car_"
const IndexName = "color~id"

func (t *APICC) tPutState(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 4 {
		return shim.Error(ErrParamNo.Error())
	}

	id := args[0]
	key := ID_Prefix + id

	var car = &Car{
		ID:         args[0],
		Color:      args[1],
		Price:      args[2],
		LaunchDate: args[3],
	}

	value, err := json.Marshal(car)
	if err != nil {
		return shim.Error(err.Error())
	}

	err = stub.PutState(key, value)
	if err != nil {
		return shim.Error(err.Error())
	}

	//创建Color与ID的组合键
	colorNameIndexKey, err := stub.CreateCompositeKey(IndexName, []string{car.Color, car.ID})
	if err != nil {
		return shim.Error("Fail to create Composite key")
	}
	stub.PutState(colorNameIndexKey, []byte{0x00}) // 将索引信息保保存在Key中

	log.Infof("new a status: %s", key)
	return shim.Success(SUCCESS)
}

func (t *APICC) queryLeveDB(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 1 {
		return shim.Error(ErrParamNo.Error())
	}
	color := args[0]
	colorIdResultsIterator, err := stub.GetStateByPartialCompositeKey(IndexName, []string{color}) //返回包含给出颜色的组合键的迭代器
	if err != nil {
        return shim.Error(err.Error())
	}
	defer colorIdResultsIterator.Close()

	var returnList []Car
	for colorIdResultsIterator.HasNext() {
		colorIdKey, err := colorIdResultsIterator.Next()
		if err != nil {
            return shim.Error(err.Error())
		}
        
        //通过SplitCompositeKey 解析出Car的主键 ID
		objectType, compisiteKeys, err := stub.SplitCompositeKey(string(colorIdKey.Key)) 

		returnColor := compisiteKeys[0]
		returnId := compisiteKeys[1]

		fmt.Printf("found a car from index %s color: %s id %s\n", objectType, returnColor, returnId)

		key := ID_Prefix + returnId
		var carItem Car
		carBytes, err := stub.GetState(key) // 根据解析出的ID获取数据
		err = json.Unmarshal(carBytes, &carItem)
		if err != nil {
			return shim.Error(fmt.Sprintf("car unmarshal err: %s", err.Error()))
		}
		returnList = append(returnList, carItem)
	}

	retJson, err := json.Marshal(returnList)
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(retJson)
}
```

## 三、历史查询

### (1) 获取迭代器

```go
func (t *SMTCC) QueryOrderHistoryByID(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 1 {
		return shim.Error(getRetString(ErrArgsNo, "expect 1"))
	}

	keyId := ID_Prefix + args[0]
	historyQueryIterator, err := stub.GetHistoryForKey(keyId)
	if err != nil {
		return shim.Error(getRetString(ErrKeyHistory, err.Error()))
	}
	defer historyQueryIterator.Close()

	history, err := getDataFromHistory(historyQueryIterator)
	if err != nil {
		return shim.Error(getRetString(ErrKeyHistory, err.Error()))
	}

	jsonByte, err := json.Marshal(history)
	if err != nil {
		return shim.Error(getRetString(ErrMarshal, err.Error()))
	}

	return shim.Success(jsonByte)
}
```

### (2) 解析迭代器

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
		itemData.

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

## 四、范围查询

### (1) 普通的范围查询

直接使用`stub.GetStateByRange(startKey, endKey)`进行查询，包含 tartKey ，不包含 end Key 

请注意，startKey和endKey可以是空字符串，这意味着在开始或结束时无限范围查询。

```go
func (t *APICC) rangeQuery(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 2 {
		return shim.Error(ErrParamNo.Error())
	}

	startKey := ID_Prefix + args[0]
	endKey := ID_Prefix + args[1]

	resultsIterator, err := stub.GetStateByRange(startKey, endKey)
	if err != nil {
		return shim.Error("Query by Range failed")
	}
	defer resultsIterator.Close() //释放迭代器

	var returnList []Car
	for resultsIterator.HasNext() {
		queryResponse, err := resultsIterator.Next() //获取迭代器中的每一个值
		if err != nil {
			return shim.Error("Fail")
		}

		// queryResponse.Key , Value, Namespace
		var carItem Car
		err = json.Unmarshal(queryResponse.Value, &carItem)
		if err != nil {
			return shim.Error("car unmarshal failed")
		}
		returnList = append(returnList, carItem)
	}

	returnJson, err := json.Marshal(returnList)
	if err != nil {
		return shim.Error("return list marshal failed.")
	}

	return shim.Success(returnJson)
}
```

### (2) 分页范围查询

```go
func (t *SMTCC) QueryOrderRangeWithPagination(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 4 {
		return shim.Error(getRetString(ErrArgsNo, "except 4."))
	}

	var startKey, endKey string
	if args[0] != "" {
		startKey = ID_Prefix + args[0]
	}

	if args[0] != "" {
		endKey = ID_Prefix + args[1]
	}

	pageSize, err := strconv.Atoi(args[2])
	if err != nil {
		return shim.Error(getRetString(ErrArgsType, "pageSize except a number string"))
	}

	bookmark := args[3]

	stateIterator, resultsBookmark, err := stub.GetStateByRangeWithPagination(startKey, endKey, int32(pageSize), bookmark)
	if err != nil {
		return shim.Error("")
	}
	defer stateIterator.Close()

	orderList, err := getDataFromState(stateIterator)
	if err != nil {
		return shim.Error(getRetString(ErrKeyHistory, err.Error()))
	}

	retObj := &RetPagination{
		Bookmark: resultsBookmark.Bookmark,
		Result:   orderList,
	}

	retByte, err := json.Marshal(retObj)
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(retByte)
}
```

## 五、富查询

只支持状态数据库，CouchDB

### (1) 查询

```go
func (t *SMTCC) QueryRich(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 1 {
		return shim.Error(getRetString(ErrArgsNo, "expect 1"))
	}

	queryString := args[0]
	stateItera, err := stub.GetQueryResult(queryString)
	if err != nil {
		return shim.Error(getRetString(ErrRich, err.Error()))
	}

	var retList []Order
	for stateItera.HasNext() {
		itemData, err := stateItera.Next()
		if err != nil {
			return shim.Error(getRetString(ErrRich, err.Error()))
		}

		var order Order
		err = json.Unmarshal(itemData.Value, &order)
		if err != nil {
			return shim.Error(getRetString(ErrArgsJson, err.Error()))
		}
		retList = append(retList, order)
	}

	retByte, err := json.Marshal(retList)
	if err != nil {
		return shim.Error(getRetString(ErrMarshal, err.Error()))
	}

	return shim.Success(retByte)
}
```



### (2) 分页查询

```go
func (t *SMTCC) QueryWithPagination(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 3 {
		return shim.Error(getRetString(ErrArgsNo, "expect 3"))
	}

	queryString := args[0]
	//return type of ParseInt is int64
	pageSize, err := strconv.Atoi(args[1])
	if err != nil {
		return shim.Error(err.Error())
	}
	bookmark := args[2]

	stateIterator, resultsBookmark, err := stub.GetQueryResultWithPagination(queryString, int32(pageSize), bookmark)
	if err != nil {
		return shim.Error(getRetString(ErrRich, err.Error()))
	}
	defer stateIterator.Close()

	orderList, err := getDataFromState(stateIterator)
	if err != nil {
		return shim.Error(getRetString(ErrKeyHistory, err.Error()))
	}

	retObj := &RetPagination{
		Bookmark: resultsBookmark.Bookmark,
		Result:   orderList,
	}

	retByte, err := json.Marshal(retObj)
	if err != nil {
		return shim.Error(err.Error())
	}

	return shim.Success(retByte)
}

```

### (3) 查询语句参数

| 参 数           | 类 型         | 描 述                                                        |
| --------------- | ------------- | ------------------------------------------------------------ |
| selector        | json          | Json对象描述用于选择文档的标准。是必备参数                   |
| limit           | number        | 返回的最大结果数。默认 25。是可选参数                        |
| skip            | number        | 路过第一个“n”结果，其中“n”是指定的值。是可选参数             |
| sort            | josn          | Json数组遵循的排序语法。是可选参数                           |
| fields          | array         | Json数组，指定每个对象的哪些字段应该返回。如果省略了，就返回整个对象。是可选参数 |
| use_index       | string\|array | 指示一个查询使用一个特定的索引。指定为“<design_document>”或["<design_document>","<index_name"]。是可选参数 |
| r               | number        | 读取结果所需的文档数量。默认为 1，在这种情况下，在索引中找到的文档会返回。如果设置为更高的值，那么每份文档至少在返回结果之前的许多副本中读取。这可能需要花费更多的时间，而不仅仅是使用索引本地存储的文档。 可选参数，默认值 1 |
| bookmark        | string        | 查询中返回，以获得下一页的结果,是可选参数                    |
| update          | boolean       | 是否在返回结果之前更新索引。默认 true。是可选参数            |
| stable          | boolean       | 视力的结果是否应该从“稳定版”一组集合中返回，是可选参数       |
| stale           | string        | update=false 和 stable=true 选项，可能赋值：“ok”。是可选参数 |
| execution_stats | boolean       | 在查询回复中包含执行统计信息，默认为 false。可选参数         |


​	



