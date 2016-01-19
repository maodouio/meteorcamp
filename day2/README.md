# 第一周周二

## Meteor基础(2)

- 日期：20160119
- 地点：线上(Moxtra)
- 讲师：田思源 & 李明

----

### Meteor 原理

#### 服务器和客户端

![](https://i.cloudup.com/J2CMCytr1Q.png)

#### 三种请求

- 静态文件
- DDP 消息
- HTTP 请求

#### 两种服务器

- HTTP Server
- DDP Server

#### MongoDB 和 Meteor

- 轮询 MongoDB 
- 使用 MongoDB oplog

### DDP 简介

DDP 是 Distributed Data Protocol 的缩写。Meteor 根据 DDP 实现了客户端和服务器端。

#### DDP是什么

是一个基于 JSON 的协议，DDP 可以在任何双向传输上实现。Meteor 的实现是基于 SockJS。

#### DDP用来做什么

它主要来做两件事：

- 处理 RPC（Remote Procedure Calls）

使用 RPC，你可以调用服务端的方法，并得到返回结果。除此之外，DDP 还可以通知调用者 method 中的所有写操作被反映到所有其他连接的客户端。

例如：客户端调用一个服务器端函数 transferMoney

![](https://i.cloudup.com/2fLpc3NA3a.png)

下面是 DDP 消息：

1. {"msg":"method", "method": "transferMoney", "params": ["1000USD", "arunoda", "sacha"], id": "randomId-1"}

2. {"msg": "result", id": "randomId-1": "result": "5000USD"}

3. {"msg": "updated", "methods": ["randomId-1"]}

解释：

1. DDP 客户端(arunoda)调用 method transferMoney ，带有三个参数：1000USD, arunoda 和 sacha；
2. 转账完成后，DDP服务端(bank)发送一条消息：arunoda的账户余额。余额在result field。如果出错了，会有一个error field；
3. 然后，DDP 服务器端发送另一条叫做 updated 的消息，带着 method id，通知我：到 sacha 的转账已经成功。有时候，updated 消息会在 result 之前返回。

- 管理数据

这是 DDP 协议的核心部分。客户端可以利用这点来订阅一个实时的数据源，并得到通知。DDP 协议有三种通知：added,changed,removed。DDP 受到 MongoDB 的启发，每一个数据通知 (一个JSON 对象) 都被赋值到一个 collection。

例如：假设数据源叫做 account，用来保存所有交易。在本例中，sacha 连接到自己的 account，获取自己的交易记录。arunoda 转账之后，sacha 会收到一个新的交易。数据流如下：

![](https://i.cloudup.com/36TF0RmTLM.png)

DDP消息如下：

1. {"msg": "sub", id: "random-id-2", "name": "account", "params": ["sacha"]}
2. {"msg": "added", "collection": "transactions", "id": "record-1", "fields": {"amount": "50USD", "from": "tom"}}
  {"msg": "added", "collection": "transactions", "id": "record-2", "fields": {"amount": "150USD", "from": "chris"}}
3. {"msg": "ready": "subs": ["random-id-2"]}
4. {"msg": "added", "collection": "transactions", "id": "record-3", "fields": {"amount": "1000USD", "from": "arunoda"}}

解释：

1. DDP 客户端(sacha)发送一个订阅请求到自己的账户；
2. 发回已发生交易的通知；
3. DDP 服务端发送完所有交易之后，会发送一个叫做ready的特殊消息。ready 消息指明订阅的所有初始数据都已经发送完成；
4. 过了一会儿，arunnoda 转账之后，sacha会收到一条added 消息。

同样的，DDP 服务端也可以发送changed 和removed通知。例如：

//changed
{"msg": "changed", collection": "transactions", "id": "doc_id", "fields": {"amount": "300USD"}}

//removed
{"msg": "removed", "collection": "transactions", "id": "doc_id"}

#### 理解和分析DDP

理解和分析 DDP 非常重要，利于你理解Meteor的工作原理。要想看到真实的 DDP 消息，可以安装 [DDP Analyzer](https://github.com/arunoda/meteor-ddp-analyzer)。

![](https://i.cloudup.com/IsUVXUOspa.png)

### 实操

### 作业

- 结合 meteor 自带的几个例子，复习已讲过的概念，试试相关的命令。

		meteor create —list

		meteor create —example [例子名]
	
- 每个人在本 repo 里最少提一个issue。