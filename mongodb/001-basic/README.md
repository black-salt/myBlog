# 001 - MongoDB 基础

<motto></motto>

## 1. 啥是 MongoDB？

## 2. 文档主键 _id

* 文档主键的唯一性
* 支持所有数据类型（数组除外）

### 2.1 对象主键 ObjectId

* 默认的文档主键
* 可以快速生成的12字节id
* 包含创建时间

## 3. 比较操作符

### 3.1 &eq 等于

### 3.2 &ne 不等于

### 3.3 &lt 小于

### 3.4 &lte 小于等于

### 3.5 &gt 大于

### 3.6 &gte 大于等于

### 3.7 &in 在筛选数组里面

匹配字段值与任一查询值相等的文档

### 3.8 &nin 不在筛选数组里面

匹配字段值与任何查询值都不等的文档

## 4. 逻辑操作符

### 4.1 $not 不

匹配筛选条件不成立的文档

### 4.2 $and 和

匹配多个筛选条件全部成立的文档

### 4.3 $or 或

匹配至少一个筛选条件成立的文档

### 4.4 $nor 都不

匹配多个筛选条件全部不成立的文档

## 5. 字段操作符

### 5.1 $exists 

匹配包含查询字段的文档

### 5.2 $type 

匹配字段类型符合查询值的文档

## 6. 数组操作符

### 6.1 $all 数组包含所有查询值

匹配数组字段中包含所有查询值的文档

### 6.2 $elemMatch 数组中有一个满足

匹配数组字段中至少存在一个值满足筛选条件的文档

## 7. 运算操作符

### 7.1 $regex 正则表达式操作符

{<field>:{:/pattern/, :'<options>'}}
{<field>:{:/pattern/<options>}}

### 7.2  $add 

### 7.3  $subtract

### 7.4  $multiply

### 7.5  $divide

### 7.6 $mod求余

## 8. 游标

游历完游标中所有的文档之后，或者在10分钟之后，游标便会自动关闭可以使用 noCursorTimeout() 函数来保持游标一直有效。

``` js
var myCursor = db.accounts.find().noCursorTimeout()
// 在这之后， 在不遍历游标的情况下， 你需要主动关闭游标
myCursor.close()
```

#### 8.1 cursor.hasNext()

#### 8.2 cursor.next()

#### 8.3 cursor.forEach()

#### 8.4 cursor.limit()

#### 8.5 cursor.skip()

#### 8.6 cursor.count()

cursor.count(<applySkipLimit>)

默认情况下，<applySkipLimit> 为 false，即 cursor.count()不会考虑cursor.skip()和cursor.limit()的效果

在不提供筛选条件时，cursor.count() 会从集合的元数据Metadata中取得结果

当数据库分布式结构较为复杂时，元数据中的文档数量可能不准确。

在这种情况下，应该避免应用不提供筛选条件的 cursor.count() 函数，而使用聚合管道来计算文档数量

#### 8.7 cursor.sort()

cursor.sort(<document>)

这里的 <document> 定义了排序的要求

{field:ordering}

1表示由小及大的正向排序，-1表示逆向排序

#### 综合使用游标函数时的问题

cursor.skip（），cursor.limit（），cursor.sort（）

cursor.skip（）在cursor.1imit（）之前执行

cursor.sort（）在cursor.skip（）和cursor.limit（）之前执行

当结合在一起使用时，游标函数的应用顺序是sort（），skip（），limit（）

## 9. 投影

### 文档投影

db.collection.find（<query>，<projection>）

不使用投影时，db.collection.find（）返回符合筛选条件的完整文档而使用投影可以有选择性的返回文档中的部分字段

{field:inclusion}

1表示返回字段，0表示不返回字段

### 在数组字段上使用投影

#### $slide

$slide操作符可以返回数组字段中的部分元素

$slide()

$elemMatch和$操作符可以返回数组字段中满足筛选条件的第一个元素

## 10. 创建 Document

### 10.1 db.collection.insertOne（）创建一个文档

``` js
db. < collection > .insertOne( <
    document > , {
        writeConcern: < document >
    }
)
// 这里的<collection>要替换成文档将要写入的集合
// 这里的<document>要替换成将要写入的文档本身
```

这里的writeconcern文档定义了本次文档创建操作的安全写级别。

简单来说，安全写级别用来判断一次数据库写入操作是否成功安全写级别越高，丢失数据的风险就越低。

然而写入操作的延迟也可能更高如果不提供writeconcern文档，mongopB使用默认的安全写级别。

``` js
db.collection.insertOne() // 返回的结果
// {"acknowledged":true，"insertedrd":"account1"}

// "acknowledged":true表示安全写级别被启用由于我们在db.collection.insertone（）命令中并没有提供writeconcern文档，这里显示的是 mongoDB 默认的安全写级别启用状态

// "insertedId“显示了被写入的文档的 _id
```

如果db.collection.insertOne() 遇到了错误 ...，使用重复的 `_id` 创建一个新文档会造成错误，可以使用 `try{} catch(e){print(e)}` 来捕捉错误并打印出错误信息。

如果需要 MongoDB 自动生成 _id，省略创建文档中的 _id 字段即可。

### 10.2 db.collection.insertMany() 创建多个文档

``` js
db. < collection > .insertMany(
    [ < document1 > ， < document2 > ，...]， {
        writeConcern: < document > ，
            ordered: < boolean >
    }
)
// ordered参数用来决定mongoDB是否要按顺序来写入这些文档
// 如果将ordered参数设置为false，mongopB可以打乱文档写入的顺序，以便优化写入操作的性能
// ordered参数的默认值为true
```

如果 db.collection.insertMany() 遇到了错误 ...

#### 在顺序写入时遇到错误

在顺序写入时，一旦遇到错误，操作便会退出，剩余的文档无论正确与否，都不会被写入。

即，如果是第一个插入便遇到错误则后面的文档均不会被插入。

如果是中间的文档插入出现错误，则不影响前面的文档的成功插入。

#### 在乱序写入时遇到错误

在乱序写入时，即使某些文档造成了错误，剩余的正确文档仍然会被写入

### 10.3 db.collection.insert() 创建单个或多个文档

``` js
db. < collection > .insert( <
    document or array of documents > ，{
        writeConcern: < document > ，
            ordered: < boolean >
    }
)

// db.collection.insert（）既可以写入一个单独的文档，也可以写入多个文档
```

### 10.4 小比较

三个命令返回的结果文档格式不一样

insertone返回的正确结果文档:

``` js
"acknowledged":
true，
    "insertedId":
    ObjectId（ "56fc40f9d735c28df206d078"）
```

insertone返回的错误结果文档:

``` js
WriteError({
    "index": 0， "code": 11000， "errmsg ": "E11000 duplicate key error collection: test.accounts index: _id dup key: {: "
    account1 "
}
"，
"op": {
    "id": "account1"，
    "name": "alice"，
    "balance": 100
}
})
```

insertMany返回的正确结果文档：

``` js
{
    "acknowledged"：
    true，
        "insertedIds"： [
            QbjectId（ "562a94d381cb9f1cd6eb0ela"），
            objectId（ "562a94d381cb9f1cd6eb0e1b"），
            ObjectId（ "562a94d381cb9f1cd6eb0elc"）
        ]
}
```

insertMany返回的错误结果文档：

``` js
BulkWriteError({
        "writeErrors": [{
            "index": 0,
            "code": 11000,
            "errmsg": "E11000 duplicate key error collection: test.accounts index: _id_ dup key: { _id: \"account1\" }",
            "op": {
                "_id": "account1",
                "name": "chy"
            }
        }],
        "writeConcernErrors": [],
        "nInserted": 0,
        "nUpserted": 0,
        "nMatched": 0,
        "nModified": 0,
        "nRemoved": 0,
        "upserted": []
    }):
    BulkWriteError({
        "writeErrors": [{
            "index": 0,
            "code": 11000,
            "errmsg": "E11000 duplicate key error collection: test.accounts index: _id_ dup key: { _id: \"account1\" }",
            "op": {
                "_id": "account1",
                "name": "chy"
            }
        }],
        "writeConcernErrors": [],
        "nInserted": 0,
        "nUpserted": 0,
        "nMatched": 0,
        "nModified": 0,
        "nRemoved": 0,
        "upserted": []
    })
```

insert写入单个文档返回的正确结果文档：

``` js
WriteResult({
    "nInserted"：
    1
})
```

insert写入单个文档返回的错误结果文档：

``` js
WriteResult({
    "nInserted"：
    0， "writeError"： {
        "code"：
        11000，
            "errmsg"：
        "insertDocument:：caused by:：11000 E11000 duplicate k ey error index:test.foo.dup key:{：1.0}"
    }
})
```

insert写入多个文档返回的正确结果文档：

``` js
BulkWriteResult({
    "writeErrors"： []，
    "writeConcernErrors"： []，
    "nInserted"：
    2， "nUpserted"：
    0， "nMatched"：
    0， "nModified"：
    0， "nRemoved"：
    0，“ upserted "：[1
})
```

insert写入多个文档返回的错误结果文档：

``` js
Bu1kWriteResult({
    "writeErrors": [{
            125542181 "code": 11000,
            "errmsg": "E11000 duplicate key error collection: test.accounts index: _id_dup key: {: account1
        }
        ",
        "op": {
            "id": "account1",
            "name": "alice",
            "balance": 100
        }
    }],
"writeConcernErrors": [],
"nInserted": 0,
"nUpserted": 0,
"nMatched": 0,
"nModified": 0,
"nRemoved": 0,
"upserted": []
})
```

### 10.5 insertone，insertMany和insert的区别

insertone 和 insertMany 命令不支持 db.collection.explain() 命令

insert 支持 db.collection.explain() 命令

### 10.6  db.collection.save()另一个可以用来创建文档的命令

``` js
db. < collection > .save( <
    document > ，{
        writeConcern： < document >
    })
```

当db.collection.save（）命令处理一个新文档的时候，它会调用db.collection.insert（）命令

## 11. 查询 Document

### 11.1 db.collection.find()

## 12. 更新 Document

### 12.1 db.collection.update()

db.<collection>.update(<query>, <update>, <options>)

### 12.2 更新操作符

#### $ set更新或新增字段

#### $ unset 删除字段

#### $ rename 重命名字段

#### $ inc 加减字段值

#### $ mul相乘字段值

#### $ min 比较减小字段值

#### $ max比较增大字段值

如果被更新的字段类型和更新值类型不一致，$min 和 $max 命令会按照 BSON 数据类型排序规则进行比较

``` js
最小 Nul1
Numbers（ ints， longs， doubles， decimals）
Symbol
String
Object
Array
BinData
ObjectId
Boolean
Date
Timestamp
最大 Regular Expression
```

### 12.3 数组更新操作符

#### $addToSet 向数组中增添元素

#### $pop 从数组中移除元素

#### $pull 从数组中有选择性地移除元素

#### $pullAll 从数组中有选择性地移除元素

#### $push 向数组中增添元素

## 13. 删除文档

### 13.1 db.collection.remove（）

db.<collection>.remove（<query>，<options>）sse1s1

<query>文档定义了删除操作时筛选文档的条件这里的<query>文档与 db.collection.find() 中的<query>文档是相同的

在默认情况下，remove命令会删除所有符合筛选条件的文档

如果只想删除满足筛选条件的*第一篇*文档，可以将 justOne 选项设置为 true

### 13.2 db.collection.drop（）

db.<collection>.drop（{writeconcern:<document>}）

这里的writeconcern文档定义了本次集合删除操作的安全写级别

之前的remove命令可以删除集合内的所有文档，但是不会删除集合

> 如果集合中的文档数量很多，使用remove命令删除所有文档的效率不高这种情况下，
> 更加有效率的方法，是使用drop命令删除集合，然后再创建空集合并创建索引
