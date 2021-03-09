# 008 - 数据故障

## 1. 响应时间长

对于一般的Web服务来说，响应时间应该在200ms以内。

对于一般的MongoDB请求来说，响应时间应该在100ms以内。

解决办法：

- 合适的索引
- 工作集超出RAM的大小，即内存不足（使用 mongostat 查看服务器状态）

## 2. 连接失败

默认情况下，mongod进程可以支持多达65536个连接。

**不恰当的配置可能限制连接数**

查看支持的连接数

`db.serverStatus().connections`

```js
# 可以在服务器的Mongodb配置文件中进行更改最大支持连接数
net: maxIncomingConnections:200
storage: 
	dbPath:/data/db 
	wiredTiger: 
		engineConfig: 
			cacheSizeGB:0.25
```















