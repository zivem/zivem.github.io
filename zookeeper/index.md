# ZooKeeper基本理解



# Zookeeper

## 简介

### 使用场景

ZooKeeper最主要的使用场景是分布式系统的分布式协同服务。

### 分布式系统服务进程通信方式

- 通过网络进行信息共享
- 通过共享存储

ZooKeeoer对分布式系统的协调，使用的是第二种方式，共享存储。

### 基本概念

ZooKeeper是一个开源的分布式协调服务，设计目标是将哪些复杂的且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一些简单的接口提供给用户使用。

ZooKeeper是一个典型的分布式数据一致性的解决方案。

基于它实现数据订阅/发布、负载均衡、命名服务、集群管理、分布式锁喝分布式队列等功能。

#### 集群

分布式系统：Master/Slave模式，主备模式

ZooKeeper：

> - Leader:通过选举选定，为客户端提供读和写服务
> - Follower：读服务
> - Observer：读服务，不参与选举

#### 会话

ZooKeeper默认端口2181.

客户端启动的时候，首先会与服务器建立一个TCP连接，从第一次建立开始，客户端生命周期开始，客户端能够心跳检测与服务器保持有效的会话，也能向Zookeeper服务器发送请求并接受响应，同时还能够通过该连接接受来自服务器的Watch事件通知。

#### 数据节点Znode

- 构成集群的机器，机器节点；
- 数据模型中的数据单元，数据节点-Znode。ZooKeeper所有数据存储在内存中，数据模型是一棵树，通过斜杆进行分割路径。

#### 版本

对于每个Znode，ZooKeeper都会为其维护一个叫做Stat的数据结构，记录了Znode的三个数据版本

- version当前版本
- cversion当前znode子节点的版本
- aversion当前znode的ACL版本，access control list 权限控制

#### Watcher事件监听器

Watcher，重要特性。zookeeper允许再指定节点注册watcher，并在一些特定事件触发

#### ACL

Zookeeper􏳔􏰉采用ACL􏴦Access ControlLists􏴧􏵣􏵤􏱆􏲴􏰪􏵥􏱁􏵦􏵠􏰌􏲢􏰜􏰝􏲧􏱭􏰳􏵧􏲾􏵥􏱁􏰞策略进行权限控制，以下五种权限：

- CREATE􏰞􏵨􏴍􏵓􏳴􏱎􏰇􏵥􏱁􏰘:创建子节点的权限
- READ􏰞􏵩􏵪􏳴􏱎􏴤􏴥􏳵􏵓􏳴􏱎􏱲􏵫􏰇􏵥􏱁􏰘:获取节点数据和子节点列表的权限
- WRITE􏰞􏵬􏵭􏳴􏱎􏴤􏴥􏰇􏵥􏱁:更新节点数据权限
- DELETE􏰞􏵮􏵯􏵓􏳴􏱎􏰇􏵥􏱁􏰘:删除子节点权限
- ADMIN􏰞􏱧􏵰:设置节点􏳴􏱎ACL􏰇􏵥􏱁􏰘权限

􏲢􏱱􏳜􏰆􏵛􏵱􏰇􏰍􏰌CREATE􏳵和Delete权限都是针对子节点的权限控制􏱁􏳍􏰍􏵲􏲌􏵓􏳴􏱎􏰇􏵥􏱁􏵦􏵠

## 基本使用

### ZNode类型

- 持久性节点
- 临时性节点
- 顺序性节点

通过组合可以生成四种节点类型：

- 持久节点

  节点创建后会一直存在服务器，直到删除操作主动清除

- 持久顺序节点

  持久性节点额外的特性，节点后就啊上一个数字后缀

- 临时节点

  会被自动清理的节点，客户端会话结束，节点会被删除，**临时节点不能创建子节点**

- 临时顺序节点

  临时节点的顺序性，添加数字后缀

### 事务ID

事务是指能够改变zookeeper服务器状态的操作，称之为事务操作或更新操作，一般包括节点创建与删除，数据节点内容更新等操作。

每一个事务请求，都会为其分配一个全局唯一的事务ID，用ZXID表示。通常是一个63位数字。每个ZXID对应一次更新操作。

### ZNode状态信息

<img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728164501307.png" alt="image-20200728164501307" style="zoom:50%;" />

#### 节点数据内容

quota节点数据

#### 状态信息

其它状态信息

### Watcher数据变更通知

分布式数据的发布/订阅功能

### ACL保障数据安全

## 应用

### 数据发布/订阅

数据发布订阅就是所谓的配置中心，发布者将数据发布到zookeeper的节点上，供订阅者进行数据订阅，今儿达到动态获取数据的目的，实现配置信息的集中式管理和数据的动态更新。

#### 设计模式

- 推Push模式，服务端主动将数据更新发送给所有订阅的客户端
- 拉Pull模式，一般客户端主动发起请求获取最新数据，采用定时轮询拉取方式

指定节点注册watcher监听，发生变更通知所有订阅的客户端

需要具备以下特性：

- 数据量小；
- 数据内容运行时发生动态变化；
- 集群中各机器共享，配置一致。

### 命名服务

#### 分布式全局唯一ID分配机制

<img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728171749706.png" alt="image-20200728171749706" style="zoom:50%;" />

基本步骤：

1. 所有客户端都会根据自己的任务类型，在指定类型的任务下通过调用crate（）接口创建一个顺序节点；
2. 节点创建完毕，crate（）接口会返回一个完整的节点名，例如：“job-000000003”；
3. 客户端拿到返回值，拼接type类型，就可以做为一个全局唯一的ID

### 集群管理

#### zookeeper两大特性：

- 客户端对数据节点watcher监听，数据节点修改，服务器向订阅的客户端发送变更通知
- 创建临时节点，一旦客户端与服务器之间的会话失效，临时节点会被删除

利用这两个特性，可以实现集群机器存活监控系统。

#### 分布式日志手机系统

核心工作：手机不同机器上的系统日志。

步骤：

1. 注册收集器机器

   > 创建一个节点做为收集器跟节点，例如/logs/collector,每台机器启动时在收集器节点下创建自己的节点

   <img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728172827712.png" alt="image-20200728172827712" style="zoom:50%;" />

2. 任务分发

   系统根据收集器节点下子节点的个数，将日志源机器分成对应的若干组，将分组后的机器列表分别写到子节点上，这样每个收集器都能从自己的节点上获取日志源机器列表，进而进行日志收集工作

3. 状态汇报

4. 动态分配

### 分布式锁

分布式锁是控制分布式系统之间同步访问共享资源的一种方式。

#### 排它锁

简称X锁，又称写锁或独占锁，是一种基本的锁类型。事务对数据对象加上了排它锁，整个加锁期间，只允许该事务对数据对象进行读取和更新操作，其他任何事务搜不能对这个数据对象做任何类型的操作。

1. ##### 定义锁

   数据节点表示一个锁

   <img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728173801986.png" alt="image-20200728173801986" style="zoom:50%;" />

2. ##### 获取锁

   调用create（）接口创建临时自己诶点，zookeeper会保证所有的客户端中，最终只有一个客户端能够创建成功，那么就可以认为该客户端获取了锁。未获取锁的在lock节点注册watcher监听。

3. ##### 释放锁

   两种方式：

   - 锁是临时节点，所以客户端届起宕机，节点会被移除，就会释放锁
   - 执行正常业务逻辑后，主动删除节点。

<img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728175740782.png" alt="image-20200728175740782" style="zoom:50%;" />

#### 共享锁

简称S锁，又称为读锁。

如果事务A对对象O加共享锁，当前事务职能对O进行读操作，其它事务也只能对这个数据对象家共享锁。

1. 定义锁

   通过数据节点表示一个锁，临时顺序节点

   <img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728175956494.png" alt="image-20200728175956494" style="zoom:50%;" />

2. 获取锁

   获取共享锁时，都会创建一个临时顺序节点。

3. 释放锁

   和独占锁一致

### 分布式队列

#### 分类

1. ##### FIFO先进先出

   创建临时顺序节点

   <img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728180423034.png" alt="image-20200728180423034" style="zoom:50%;" />

   > 执行顺序：
   >
   > 1. 获取所有元素
   > 2. 确定自己的节点序号在子节点中的顺序
   > 3. 如果自己的序号不是最小，那么需要等待，同时比自己序号小的最后一个节点注册watcher监听
   > 4. 接收到watcher通知，重复第一步
   >
   > <img src="/assets/img/post/2020-07-28-Zookeeper/image-20200728180439127.png" alt="image-20200728180439127" style="zoom:50%;" />

2. #### Barrier：分布式屏障

   指分布式系统之间的一个协调条件，规定了一个队列元素必须都集聚后才能统一进行安排，否则一直等待。

   

