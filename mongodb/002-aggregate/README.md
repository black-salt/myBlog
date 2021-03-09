# 002 - MongoDB 聚合管道

<motto></motto>

## 1. 聚合操作

### db.collection.aggregate（）

db.<collection>.aggregate（<pipeline>，<options>）

<pipeline>文档定义了操作中使用的聚合管道阶段和聚合操作符

<options>: (有很多，就只说一个 allowDiskUse)

allowDiskUse：<boolean>每个聚合管道阶段使用的内存不能超过100MB如果数据量较大，为了防止聚合管道阶段超出内存上限并且抛出错误，可以启用allowDiskUse选项

allowDiskUse启用之后，聚合阶段可以在内存容量不足时，将操作数据写入临时文件中

临时文件会被写入dbPath下的_tmp文件夹，dbpath的默认值为/data/db

## 2. 聚合操作符

| 表达式    | 描述                                           | 实例                                                         |
| :-------- | :--------------------------------------------- | :----------------------------------------------------------- |
| $sum      | 计算总和。                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 在结果文档中插入值到一个数组中。               | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 在结果文档中插入值到一个数组中，但不创建副本。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。         | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据         | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

## 3. 聚合表达式

介绍几种常见的表达式

### 字段路径表达式

$<field> - 使用 $ 来指示字段路径

$<field>.<sub-field> - 使用 $ 和 . 来指示内嵌文档字段路径

$ name - 指示银行账户文档中客户姓名的字段

$ info.dateOpened - 指示银行账户文档中开户日期的字段

### 系统变量表达式

$$<variable> - 使用$$来指示系统变量

$$CURRENT - 指示管道中当前操作的文档 - $$CURRENT.<field> 和 $<field>是等效的

### 常量表达式

$literal：<value> - 指示常量<value>

$literal:"$name" - 指示常量字符串"$name" - 这里的$被当作常量处理，而不是字段路径表达式

### 聚合管道阶段

$project - 对输入文档进行再次投影
$match - 对输入文档进行筛选
$limit - 筛选出管道内前N篇文档
$skip - 跳过管道内前N篇文档
$unwind - 展开输入文档中的数组字段
$sort - 对输入文档进行排序
$lookup - 对输入文档进行查询操作
$group - 对输入文档进行分组
$out - 将管道中的文档输出

### $project - 对输入文档进行再次投影

$project是一个很常用的聚合阶段可以用来灵活地控制输出文档的格式也可以用来剔除不相关的字段，以优化聚合管道操作的性能

### $match - 对输入文档进行筛选

$match中使用的文档筛选语法，和**读取文档**时的筛选语法相同

$match也是一个很常用的聚合阶段

应该尽量在聚合管道的开始阶段应用$match

这样可以减少后续阶段中需要处理的文档数量，优化聚合操作的性能

### $limit - 筛选出管道内前N篇文档

这个就不说了，直接加数字就好了

### $skip - 跳过管道内前N篇文档

这个就不说了，直接加数字就好了

### $unwind - 展开输入文档中的数组字段

``` js
// “展开数组时添加元素位置"
db.accounts.aggregate([{
    $unwind: {
        path: "$currency"，
        includeArrayIndex: "ccyIndex"
    }
}])

// “展开数组时保留空数组或不存在数组的文档”
db.accounts.aggregate([{
    $unwind: {
        path: "$currency"，
        preserveNullAndEmptyArrays: true
    }
}])
```

### $sort - 对输入文档进行排序

和查询时的排序一样，

对某个字段正序排列输入 1，

反序排列输入 -1

### $lookup - 对输入文档进行查询操作

``` js
// 使用单一字段值进行查询
$lookup: {
    from： < collection to join > , //同一数据库中的另一个查询集合
    localField： < field from the input documents > , //管道文档中用来进行查询的字段
    foreignField： < field from the documents of the "from"
    collection > , //查询集合中的查询字段
    as： < output array field > //写入管道文档中的查询结果数组字段
}

// 如果localField是一个数组字段,也可以进行查询，新字段匹配到多个的话返回的也会是数组，
// 如果想要分离开的话可以使用unwind在lookup管道前进行分离。
```

``` js
// 使用复杂条件进行查询
$lookup: {
    from： < collection to join > ,
    let: {
        <
        var_1 > ： < expression > ，...， < var_n > ： < expression >
    }, //对查询集合中的文档使用聚合阶段进行处理时，如果需要参考管道文档中的字段，则必须使用let参数对字段进行声明
    pipeline: [ < pipeline to execute on the collection to join > ], //对查询集合中的文档使用聚合阶段进行处理
    as： < output array field > //写入管道文档中的查询结果数组字段
}

db.accounts.aggregate([{
    $lookup: {
        from: "forex",
        pipeline: [{
                $match: {
                    date: new Date("2018-12-21")
                ],
                as: "forexData"
            }
        }
    }
}])
// 注意，在这个例子中，查询条件和管道文档之间，其实并没有直接的联系这种查询被称作不相关查询，$lookup从3.6版本开始支持不相关查询

db.accounts.aggregate([
    $lookup: {
        from: "forex",
        let: {
            bal: "$balance"
        },
        pipeline: [{
                $match: {
                    $expr: {
                        $and: [{
                                $eq: ["$date", new Date("2018-12-21")]
                            },
                            {
                                $gt: ["$$bal", 100]
                            }
                        ]
                    }
                }
            ]
        },
        as: "forexData"
    }
])
```

### $group - 对输入文档进行分组

``` js
$group: {
    id： < expression > , // 定义分组规则
    <field1 > ：{
        < accumulator1 > ： < expression1 > // 可以使用聚合操作符来定义新字段
    },
    ...
}
```

* 不使用聚合操作符的情况下，$group可以返回管道文档中某一字段的所有（不重复的）值。
* 使用聚合操作符的情况下，$group可以返回管道文档中某一字段的所有（不重复的）值的类下的统计相关数据。
* 使用聚合操作符计算**所有文档聚合值**时，可以给 _id 赋值为null

### $out - 将管道中的文档输出

$out 可以将聚合管道中的文档写入一个新集合

如果聚合管道操作遇到错误，$out 管道阶段不会创建新集合或是覆盖已存在的集合内容

## 4. 聚合操作的优化

### 4.1 聚合阶段顺序优化

#### 1. $project + $match

$match 阶段会在 $project 阶段之前运行。

$match 是可以一分为二或者更多的。因为有的字段可以提前有的不可以。（$match 中筛选条件不受 $project 影响的话，那么该筛选条件就是可以提前的）

#### 2. $sort + $match

$match 阶段会在 $sort 阶段之前运行

#### 3. $project + $skip

$skip 阶段会在 $project 阶段之前运行

### 4.2 聚合阶段合并优化

#### 1. $sort +$limit 

如果两者之间没有夹杂着会改变文档数量的聚合阶段，$sort 和 $limit 阶段可以合并。

比如 $sort 和 $limit 之间夹杂了 $project ，那么 $limit 可以提到前面来和 $sort进行合并。

#### 2. $limit +$limit，$skip + $skip，$match +$match

$limit +$limit，$skip + $skip，$match +$match连续的$limit，$skip或$match阶段排列在一起时，可以合并为一个阶段。

#### 3. $lookup+ $unwind

连续排列在一起的$1ookup和￥unwind阶段，如果$unwind应用在$lookup阶段创建的 as 字段上，则两者可以合并。 

## 5. 总结

使用db.collection.aggregate（）命令进行聚合操作
使用聚合表达式
常用的聚合管道阶段
常用的聚合操作符
聚合操作的局限和优化
