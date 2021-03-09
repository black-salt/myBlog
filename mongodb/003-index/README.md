# 003 - MongoDB 索引

<motto></motto>

>  合适的索引可以大大提升数据库搜索性能

复合键索引只能支持前缀子查询

- 对文档部分内容进行排序的数据结构
- 加快文档查询和文档排序的速度
- 复合键索引只能支持前缀子查询

索引操作：

- db.collection.getlndexes() 列出集合中已存在的索引
- db.collection.createlndex() 创建索引
- db.collection.dropIndex() 删除索引

索引的类型:

- 单键索引
- 复合键索引
- 多键索引

索引的特性：

- 唯一性
- 稀疏性
- 生存时间

查询分析：

- 检视索引的效果
- explain()

索引的选择：

- 如何创建一个合适的索引
- 索引对数据库写入操作的影响

## 1. 创建索引

```js
db.collection.createIndex()
db.<collection>.createIndex(<keys>，<options>)
<keys>文档指定了创建索引的字段，1 代表该字段正向排序，-1 代表该字段逆向排序。
```

如果对数组字段创建索引，则数组字段中的每一个元素，都会在多键索引中创建一个键

## 2. 索引的效果

```js
db.collection.explain()
db.<collection>.explain().<method(...)>
可以使用explain()进行分析的命令包括aggregate()，count()，distinct()，find()，group()，remove()，update()
```

### 2.1 查找效果

使用没有创建索引的字段进行搜索

```js
> db.accountsWithIndex.explain().find({type:"saving" })
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.accountsWithIndex",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "type" : {
                                "$eq" : "saving"
                        }
                },
                "queryHash" : "2A1623C7",
                "planCacheKey" : "2A1623C7",
                "winningPlan" : {
                        "stage" : "COLLSCAN",// 查询方式，最慢的查询
                        "filter" : {
                                "type" : {
                                        "$eq" : "saving"
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "DESKTOP-PIH1O9K",
                "port" : 27017,
                "version" : "4.2.0",
                "gitVersion" : "a4b751dcf51dd249c5865812b390cfd1c0129c30"
        },
        "ok" : 1
}
```

使用有创建索引的字段进行搜索

```js
> db.accountsWithIndex.explain().find({name:"lskreno"})
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.accountsWithIndex",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "name" : {
                                "$eq" : "lskreno"
                        }
                },
                "queryHash" : "01AEE5EC",
                "planCacheKey" : "0BE5F32C",
                "winningPlan" : {
                        "stage" : "FETCH",// 到索引对应的地址进行获取
                        "inputStage" : {
                                "stage" : "IXSCAN",// 查询方式
                                "keyPattern" : {
                                        "name" : 1
                                },
                                "indexName" : "name_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "name" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "name" : [
                                                "[\"lskreno\", \"lskreno\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [
                        {
                                "stage" : "FETCH", 
                                "inputStage" : {
                                        "stage" : "IXSCAN", 
                                        "keyPattern" : {
                                                "name" : 1,
                                                "balance" : -1
                                        },
                                        "indexName" : "name_1_balance_-1",
                                        "isMultiKey" : false,
                                        "multiKeyPaths" : {
                                                "name" : [ ],
                                                "balance" : [ ]
                                        },
                                        "isUnique" : false,
                                        "isSparse" : false,
                                        "isPartial" : false,
                                        "indexVersion" : 2,
                                        "direction" : "forward",
                                        "indexBounds" : {
                                                "name" : [
                                                        "[\"lskreno\", \"lskreno\"]"
                                                ],
                                                "balance" : [
                                                        "[MaxKey, MinKey]"
                                                ]
                                        }
                                }
                        }
                ]
        },
        "serverInfo" : {
                "host" : "DESKTOP-PIH1O9K",
                "port" : 27017,
                "version" : "4.2.0",
                "gitVersion" : "a4b751dcf51dd249c5865812b390cfd1c0129c30"
        },
        "ok" : 1
}
```

仅返回创建了索引的字段

```js
> db.accountsWithIndex.explain().find({name:"lskreno" },{_id:0,name:1})
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.accountsWithIndex",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "name" : {
                                "$eq" : "lskreno"
                        }
                },
                "queryHash" : "2E50AAA9",
                "planCacheKey" : "BAD68206",
                "winningPlan" : {
                        "stage" : "PROJECTION_COVERED",// 无需去地址里取字段，直接从索引里返回
                        "transformBy" : {
                                "_id" : 0,
                                "name" : 1
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",// 查询方式
                                "keyPattern" : {
                                        "name" : 1,
                                        "balance" : -1
                                },
                                "indexName" : "name_1_balance_-1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "name" : [ ],
                                        "balance" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "name" : [
                                                "[\"lskreno\", \"lskreno\"]"
                                        ],
                                        "balance" : [
                                                "[MaxKey, MinKey]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [
                        {
                                "stage" : "PROJECTION_COVERED",
                                "transformBy" : {
                                        "_id" : 0,
                                        "name" : 1
                                },
                                "inputStage" : {
                                        "stage" : "IXSCAN",
                                        "keyPattern" : {
                                                "name" : 1
                                        },
                                        "indexName" : "name_1",
                                        "isMultiKey" : false,
                                        "multiKeyPaths" : {
                                                "name" : [ ]
                                        },
                                        "isUnique" : false,
                                        "isSparse" : false,
                                        "isPartial" : false,
                                        "indexVersion" : 2,
                                        "direction" : "forward",
                                        "indexBounds" : {
                                                "name" : [
                                                        "[\"lskreno\", \"lskreno\"]"
                                                ]
                                        }
                                }
                        }
                ]
        },
        "serverInfo" : {
                "host" : "DESKTOP-PIH1O9K",
                "port" : 27017,
                "version" : "4.2.0",
                "gitVersion" : "a4b751dcf51dd249c5865812b390cfd1c0129c30"
        },
        "ok" : 1
}
```

### 2.2 排序效果

使用已经创建索引的字段进行排序

```js
> db.accountsWithIndex.explain().find().sort({name: 1,balance:-1})
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.accountsWithIndex",
                "indexFilterSet" : false,
                "parsedQuery" : {

                },
                "queryHash" : "DC9EFEDE",
                "planCacheKey" : "DC9EFEDE",
                "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "name" : 1,
                                        "balance" : -1
                                },
                                "indexName" : "name_1_balance_-1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "name" : [ ],
                                        "balance" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "name" : [
                                                "[MinKey, MaxKey]"
                                        ],
                                        "balance" : [
                                                "[MaxKey, MinKey]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "DESKTOP-PIH1O9K",
                "port" : 27017,
                "version" : "4.2.0",
                "gitVersion" : "a4b751dcf51dd249c5865812b390cfd1c0129c30"
        },
        "ok" : 1
}
```

使用未创建索引的字段进行排序

```js
> db.accountsWithIndex.explain().find().sort({name: 1,balance:1})
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.accountsWithIndex",
                "indexFilterSet" : false,
                "parsedQuery" : {

                },
                "queryHash" : "797A24CD",
                "planCacheKey" : "797A24CD",
                "winningPlan" : {
                        "stage" : "SORT", // 很耗费资源的PLAN
                        "sortPattern" : {
                                "name" : 1,
                                "balance" : 1
                        },
                        "inputStage" : {
                                "stage" : "SORT_KEY_GENERATOR",
                                "inputStage" : {
                                        "stage" : "COLLSCAN",
                                        "direction" : "forward"
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "DESKTOP-PIH1O9K",
                "port" : 27017,
                "version" : "4.2.0",
                "gitVersion" : "a4b751dcf51dd249c5865812b390cfd1c0129c30"
        },
        "ok" : 1
}
```

## 3. 删除索引

db.collection.dropIndex（）

如果需要更改某些字段上已经创建的索引

必须首先删除原有索引，再重新创建新索引

否则，新索引不会包含原有文档

### 3.1 使用索引名称删除索引

db.accountswithIndex.dropIndex("name_1")

### 3.2 使用索引定义删除索引

db.accountswithIndex.dropIndex({"name":1,"balance":-1})

### 3.3 使用索引名称删除索引



## 4. 索引的特性

创建索引，我们使用 db.collection.createIndex（）

db.<collection>.createIndex（<keys>，<options>）

<options>文档定义了创建索引时可以使用的一些参数

<options>文档也可以设定索引的特性

### 4.1 索引的唯一性 {unique: true}

文档主键上创建的默认索引

创建一个具有唯一性的索引

```js
db.accountswithIndex.createIndex({balance：1}，{unique:true})
```

`unique:true` 强制保证了插入的 balance必须唯一

- 如果已有文档中的某个字段出现了重复值，就不可以在这个字段上创建唯一性索引
- 如果强行创建`unique:true`则报错。
- 如果新增的文档不包含唯一性索引字段，只有*第一篇*缺失该字段的文档可以被写入数据库，索引中该文档的键值被默认为 null
- 复合键索引也可以具有唯一性，在这种情况下，*不同的*文档之间，其所包含的复合键字段值的组合，不可以重复

### 4.2 索引的稀疏性 {sparse: true}

只将包含索引键字段的文档加入到索引中（即使索引键字段值为null）

```js
db. accountswithIndex. createIndex({ balance:1},{ sparse: true})
```

- 如果同一个索引既具有`唯一性`，又具有`稀疏性`，就可以保存*多篇*缺失索引键值的文档了
- 复合键索引也可以具有稀疏性，在这种情况下，只有在缺失复合键所包含的所有字段的情况下，文档才不会被加入到索引中"

### 4.3 索引的生存时间 {expireAfterSeconds: 20} 20指20秒

针对日期字段，或者包含日期元素的数组字段，可以使用设定了生存时间的索引，来自动删除字段值超过生存时间的文档

- 复合键索引*不*具备生存时间特性
- 当索引键是包含日期元素的数组字段时，数组中*最小*的日期将被用来计算文档是否已经过期
- 数据库使用一个后台线程来监测和删除过期的文档，删除操作可能有一定的延迟

