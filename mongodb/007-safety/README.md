# 007 - 数据安全

## 1. 数据库身份认证

### 1.1 创建第一个用户

使用mongo she11创建用户

``` js
use admin；
db.createUser({
    user: "userAdmin",
    pwd: "useradmin",
    roles: ["userAdminAnyDatabase"]
})
```

### 1.2 启用身份认证

重启 mongod 进程后, 并且启动时加上 `-auth` 参数, 在使用客户端时才会进行用户验证。

#### 身份验证的方法：

1. 使用用户名和密码进行身份验证

`$ mongo -u "userAdmin" -p "useradmin" --authenticationDatabase "admin"` 

2. 使用db.auth（）进行身份验证（也就是说不在启用 mongo shell 时验证用户身份, 而是在匿名的情况下执行 `db.auth()函数` 验证）

`> use admin;` 

`db.auth("userAdmin","useradmin")` 

## 2. 身份授权 

### 权限

权限 = 在哪里 + 做什么

``` js
{
    resource: {
        db: "test",
        collection: ""
    },
    actions: ["find",
        "update"
    ]
} {
    resource: {
        cluster: true
    },
    actions: ["shutdown"]
}
```

角色
角色 = 一组权限的集合
read — 读取当前数据库中所有非系统集合

readWrite — 读写当前数据库中所有非系统集合

dbAdmin — 管理当前数据库

userAdmin — 管理当前数据库中的用户和角色

read/readwrite/dbAdmin/userAdmin + AnyDatabase — 对所有数据库执行操作（只在ad
min数据库中提供）

``` js
# 创建一个只能读取test数据库的用户

use test；
db.createUser({
    user: "testReader",
    pwd: "passwd",
    roles: [{
        role: "read",
        db: "test"
    }]
})
```

## 3. 自定义角色

``` js
# 创建一个只能读取accounts集合的用户

use test;
db.createRole({
    role: "readAccounts",
    privileges: [{
        resource: {
            db: "test",
            collection: "accounts"
        },
        actions: ["find"]
    }],
    roles: []
})
```

## 4. 数据管理

### mongoexport

### mongoimport

## 5. 数据库状态监控

### mongostat
显示数据库服务器进程状态

需要对操作的数据库具备 clusterMonitor 角色的权限

``` js
# 创建执行mongostat的用户使用mongo she11创建用户
use admin;
db.createUser({
    user: "monitorUser",
    pwd: "passwd",
    roles: ["clusterMonitor"]
})
```

``` js
# 显示数据库进程状态
mongostat--host localhost--port 27017 - u monitorUser - p passwd--authenticationDatabase admin
# 每隔3秒报告一次状态
mongostat--host localhost--port 27017 - u monitorUser - p passwd--authenticationDatabase admin 3
# 限制报告状态的次数
mongostat--host localhost--port 27017 - u monitorUser - p passwd--authenticationDatabase admin--rowcount 5 3
# 有选择地显示状态
mongostat--host localhost--port 27017-u monitorUser-p passwd--authenticationpatabase admin-o"command,dirty,used,vsize,res,conn,time"
```
command — 每秒执行的命令数

dirty, used — 数据库引擎缓存的使用量百分比（他们应该相差不多，相差过多时一般是缓存不够了）

vsize — 虚拟内存使用量（MB）

res — 常驻内存使用量（MB）

conn — 连接数

### mongotop

显示各个集合上的读写时间

需要对操作的数据库具备 clusterMonitor 角色的权限

```js
# 显示各个集合上的读写时间
mongotop--host localhost--port 27017-u monitorUser-p passwd--authenticationpatabase admin
# 每隔3秒报告一次状态
mongotop--host localhost--port 27017-u monitorUser-p passwd--authenticationpatabase admin 3
# 限制报告状态的次数
mongotop--host localhost--port 27017-u monitorUser-p passwd--authenticationpatabase admin--rowcount53
```



















