# YARN

## 1. 架构

`Yarn` 的基本思想是为了切分 `资源管理` 和 `任务调度/监控` 为几个守护进程来执行，在 `Hadoop 1.x` 版本中，这些工作都由 `MapReduce` 负责。`Hadoop 2.x` 后，引入了 `ResouceManager` 、 `NodeManager` 、 `ApplicationMaster` 和 `Container` 来负责。

* `ResourceManager`职责
  * 处理客户端请求
  * 监控 `NodeManager`
  * 启动和监控 `ApplicationMaster`
  * 资源分配与调度
* `NodeManager`职责
  * 管理每个节点上的资源
  * 处理来自 `ResourceManager` 的命令
  * 处理来自 `ApplicationMaster` 的命令
* `ApplicationMaster` 职责
  * 负责数据的切分
  * 为应用程序申请资源并分配给对应的任务
  * 任务的监控和容错

![](yarn_architecture.gif)

### 

## 2. 任务执行流程

1. 客户端程序提交任务到客户端所在的节点
2. `YarnRunner Client` 向 `ResourceManager`申请创建 `Application`
3. `ResourceManager` 返回 `Application`资源路径给 `YanrRunner Client`
4. 客户端程序将运行所需的信息和数据提交到 `HDFS`
5. 客户端程序申请运行 `MRAppMaster`
6. `ResourceManager` 将用户的请求初始化为一个 `Task`
7. `NodeManager` 领取 `Task` 任务
8. `NodeManager` 创建容器 `Container`，并产生 `MRAppRuner`
9. `Container` 从 `HDFS` 上拷贝资源到本地
10. `MRAppMaster` 向 `ResourceManager` 申请运行 `MapTask` 资源
11. `ResourceManager` 将运行 `MapTask` 任务分配给另外两个 `NodeManager`, 另两个 `NodeManager` 分别领取任务并创建容器
12. ``

## 3. 调度器

`Yarn` 当前实现的调度器有 `FIFO`、`Capacity Scheduler` 和 `Fair Scheduler`

### 2.1 FIFO调度器

先进先出的队列调度器，运行原理如下图所示

### 2.1 容量调度器

基于 `FIFO` 基础上，添加了组的概念，每个组有资源容量限制，每个组内是一个 `FIFO`队列，运行原理如下图所示



### 2.2 公平调度器

依据资源配比，公平分配资源给各个任务，资源缺额越大，分配到资源越多。运行原理如下图所示

#### 4. 