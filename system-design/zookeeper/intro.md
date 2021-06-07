# Zookeeper 入门

## 1 概述

Zookeeper 是一个开源的分布式的，为 `分布式应用提供协调服务` 的 Apache 项目。

## 2 基本概念

1. `ZooKeeper`: 一个`Leader`, 多个 `Follower` 组成的集群；
2. 集群中只要有半数以上的节点存活(如有2n+1个节点的集群，只要n+1个节点存活), `ZooKeeper` 就能正常服务；
3. 全局数据一致：每个`ZooKeeper Server`都保存着一份相同的数据副本，`Client` 无论连接到哪个 `Server` ，获取的数据都是一致的；
4. 数据更新按请求顺序执行： 来自 `Client`的更新请求，都会按照它的发送的顺序来依次执行；
5. 数据更新原子性：一次数据更新，要么成功，要么失败；
6. 实时性：在一定范围内，`Client` 能读到最新的数据；

## 3 数据结构

`Zookeeper` 数据模型的结构与`Unix`文件系统很类似，整体上可以看做是一棵树，每个节点称作一个`ZNode`。每一个`ZNode`默认能够存储`1MB`的数据，每个`ZNode`都可以通过其路径唯一标识。

![](zk-data-structure.jpg)

## 4 应用场景

1. `统一命名服务`: 分布式环境下，经常需要对应用/服务进行统一命名，便于识别。
2. `统一配置管理`: 分布式环境中，集群的所有节点的配置信息要求是一致的，且配置信息修改后，我们希望所有节点都可以快速同步到最新的配置信息；这时我们可以将配置信息写入`ZNode`,然后服务监听该`ZNode`,那么一旦配置信息被修改，服务可以马上获取到修改通知；
3. `统一集群管理`: 分布式环境下，需要实时获取同步各个节点的运行状态，这时我们可以定时将节点的运行状态同步到`ZNode`中，服务监听该`ZNode`获取实时的节点运行状态信息；
4. `服务节点动态上下线`: 类似`Nginx`，当后端节点不可达时，不再转发请求到该节点；同理，`ZNode`中存储了实时的节点运行状态，服务只需要在请求后端时
5. `负载均衡`: 在 4 的基础上，增加对服务器访问数的记录，这样在分配请求路由时选择当前访问数最少的服务器进行路由转发，则实现了负载均衡的功能；

## 5 安装

### 5.0 系统要求
1. 当前`ZooKeeper`服务支持在`GNU/Linux`、`Solaris`、`FreeBSD`、`Windows`中进行正式运行环境和开发环境，其中`Mac OS X`中支持开发模式；
2. `ZooKeeper`运行在Java虚拟机上，要求`JDK 1.8`或以上(`JDK 8 LTS`, `JDK 11 LTS`, `JDK 12` - 其中`Java 9` 和 `Java 10` 不支持)。
3. `ZooKeeper`推荐使用双核处理器，2GB RAM和80GB的硬盘容量;


### 5.1 本地模式安装
1. 下载安装文件，请下载稳定版本，即官网指明处于`stable`的安装包版本；[下载地址](https://zookeeper.apache.org/releases.html)
2. 下载安装包后，进行解压
    ```
    tar xzvf apache-zookeeper-3.6.2-bin.tar.gz ~/bigdata/
    ```
3. 修改 `conf/zoo.cfg` 配置文件
    ```sh
    # ZooKeeper 中时间的基本单位是毫秒，tickTime用于服务器与客户端的心跳时间间隔，
    # 还有回话的保持时间是 tickTime 的两倍
    tickTime=2000
    # 用于存储 ZooKeeper 的数据库快照和数据库的事务日志
    dataDir=/var/lib/zookeeper
    # ZooKeeper 监听的端口
    clientPort=2181
    ```
4. 运行
    ```
    bin/zkServer.sh start
    ```
5. 连接
    ```
    bin/zkCli.sh -server 127.0.0.1:2181
    ```
6. 命令测试，输入`help`命令即可查看`ZooKeeper`客户端中支持的命令
   ```
   [zk: localhost:2181(CONNECTED) 1] help
    ZooKeeper -server host:port -client-configuration properties-file cmd args
	addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
	addauth scheme auth
	close
	config [-c] [-w] [-s]
	connect host:port
	create [-s] [-e] [-c] [-t ttl] path [data] [acl]
	delete [-v version] path
	deleteall path [-b batch size]
	delquota [-n|-b] path
	get [-s] [-w] path
	getAcl [-s] path
	getAllChildrenNumber path
	getEphemerals path
	history
	listquota path
	ls [-s] [-w] [-R] path
	printwatches on|off
	quit
	reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
	redo cmdno
	removewatches path [-c|-d|-a] [-l]
	set [-s] [-v version] path data
	setAcl [-s] [-v version] [-R] path acl
	setquota -n|-b val path
	stat [-w] path
	sync path
	version
    ```

### 5.2 分布式安装 
1. 准备三台机器(`ZooKeeper`安装推荐使用`2n+1`个实例),假设为`zk001`、`zk002`、`zk003`；
2. 同`本地模式安装`中的`1、2`步骤；
3. 在 `/var/lib/zookeeper`(数据存放目录，随自己配置修改) 下，创建`myid`文件，各台机器下分别输入`1`、`2`、`3`，表示各个实例的ID；
4. 创建 `zoo.cfg` 文件，从`zoo_sample.cfg`文件中拷贝而来；
5. 修改 `zoo.cfg` 文件
    ```sh
    dataDir=/var/lib/zookeeper
    # 初始通信时，Follower 连接到 Leader 的最大容忍次数
    initLimit=5
    # 同步通信时，Follower 与 Leader 的x最大容忍响应次数，若超过 tickTime * syncLimit 时，
    # Leader 仍未收到 Follower 的响应，则认为该 Follower 已宕机；
    syncLimit=2
    # 增加如下配置
    # 对于 server.ID=hostname:port1:port2
    # ID 为 第3步 中配置的实例ID
    # hostname为实例的hostname或者IP
    # port1 为 Follower 与 Leader 通信端口
    # port2 为 执行Leader选举时，实例间的通信端口
    server.1=zk001:2888:3888
    server.2=zk002:2888:3888
    server.3=zk003:2888:3888
    ```
6. 启动集群，在所有集群上启动 `ZooKeeper Server` 服务
    ```
    bin/zkServer.sh start
    ```
7. 查看集群状态
    ```
    bin/zkServer.sh status
    ```

## 6 常用客户端命令

以下命令中，flag的作用
- `-s`: 有序节点
- `-w`: 监听节点
- `-e`: 临时节点
- `-R`: recursive，递归
- `-v`: 指定 version
- `acl`: 设置节点的 ACL 属性
- `-t ttl`: 指定 ttl 时间

| 命令                                             | 功能                        |
| :----------------------------------------------- | :-------------------------- |
| help                                             | 显示所有命令及其帮助信息    |
| ls [-s] [-w] [-R] path                           | 查看 path 下中包含的内容，  |
| create [-s] [-e] [-c] [-t ttl] path [data] [acl] | 创建 path 节点              |
| get [-s] [-w] path                               | 获取 path 节点的信息        |
| set [-s] [-v version] path data                  | 设置 path 节点的数据为 data |
| stat [-w] path                                   | 获取 path 节点的属性信息    |
| delete [-v version] path                         | 删除 path 节点              |

## 7 Stat 结构体