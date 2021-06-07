# 1. Hadoop 概述

## 1.1 Hadoop 是什么

Hadoop最早起源Nutch,它的设计目标是构建一个大型的全网搜索引擎，包括网页抓取，索引，查询等功能，但由于庞大的 `数据量`，上亿级的，遇到了 `可扩展性` 问题;

2003开始年谷歌陆续发表了两篇论文成为可行性解决方案:
- `分布式文件系统(GFS)`用于处理海量数据存储
- `分布式计算框架MapReduce`用于处理海量网页的索引计算问题

Nutch开发人员完成了相应的开源实现 `HDFS` 和 `MapReduce`，并从Nutch中剥离成为独立项目 `Hadoop`

## 1.2 Hadoop 能做什么？

由1.1中可以看出，`Hadoop` 主要包含以下组件

- `HDFS`: 解决海量数据的存储；
- `MapReduce`: 解决海量数据的分布式计算；
- `Yarn`: 2.0版本引入，原本包含在 `MapReduce` 的 `JobTracker` 中；它是一个资源管理框架，负责资源的管理和调度；

所以，`Hadoop` 就是为了解决海量数据的存储和分布式计算问题而来的，现已广泛应用于各种大数据处理场景。
## 1.3 Hadoop 的优势

- 高可靠：Hadoop底层(依靠HDFS)维护多个数据副本，某个数据或机器的故障，并不会导致数据的丢失；
- 高扩展：通过副本和数据块，在集群内分配数据和执行任务，集群节点可以扩展到数以万计；
- 高效： 通过 `MapReduce` 支持下，Hadoop的任务都是并行执行的，任务处理的速度快；
- 高容错： 任务失败时，能自动将失败的任务重新分配到其他机器执行；

## 1.4 Hadoop 生态圈

- 狭义上，`Hadoop` 由 `HDFS`、`MapReduce`和 `Yarn` 三个组件组成的一个大数据存储和分析的系统；
- 广义上, 泛指大数据技术生态圈，包括 `Hadoop`、`Zookeeper`、`Kafka`、`Hive`、`Flume`、`HBase`、`Spark`和`Flink`等其他大数据相关组件；

![](bigdata-system.jpg)

我们接下来说的 `Hadoop` 都是狭义上的意义。

## 1.5 Hadoop 各版本区别
- 1.X，主要有`MapReduce`、`HDFS`、`Common`；
- 2.X
  * 在1.X的基础上，从 `MapReduce` 中抽离资源的管理和调度功能，由 `Yarn`实现；
  * `HDFS` 中添加 `standbynamenode`,解决1.X的`namenode`单点故障问题；
- 3.X
  * JVM 升级为 1.8；
  * HDFS 支持纠删码；
  * Yarn时间线服务
  * 支持2个以上的 `namenode`;
  * `MapReduce` 本地化优化，性能提升 30%；


# 2. Hadoop 安装

## 2.1 安装JDK

```sh
apt update
apt install -y openjdk-11-jdk
java -version

echo 'JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' > /etc/profile
```
## 2.2  安装单机Hadoop
  - 2.2.1 下载安装包

  - 2.2.2 上传安装包
    ```sh
    mkdir -p /opt/software
    sftp root@ubuntu001:/opt/software Downloads/Sofewares/hadoop-3.2.2.tar.gz 
    ```

  - 2.2.3 解压
    ```sh
    mkdir -p /opt/module
    tar -xzvf /opt/software/hadoop-3.2.2.tar.gz -C /opt/module
    cd /opt/module && mv hadoop-3.2.2 hadoop
    ```

  - 2.2.4 配置 HADOOP_HOME
    在 /etc/profile 文件末尾添加下列内容
    ```sh
    export HADOOP_HOME=/opt/module/hadoop
    export PATH=$PATH:$HADOOP_HOME/bin
    export PATH=$PATH:$HADOOP_HOME/sbin
    ```

  - 2.2.5 验证安装
    
    查看版本
    ```sh
    hadoop version
    ```

    官方案例: 将 etc/hadoop/conf 下的 *.xml 文件拷贝到 input 作为输入，从中查找与正则表达式 `dfs[a-z.]+` 匹配的字符串，并将结果输出到 output

    ```sh
    mkdir input
    cp etc/hadoop/*.xml input
    bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.2.jar grep input output 'dfs[a-z.]+'
    cat output/*
    ```

    wordcount案例：
    ```sh
    mkdir wcinput

    echo `
      hello world
      hello hadoop
      hello yarn
      hello hdfs
      hello mapreduce
    ` > wcinput/wc.txt
    hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.2.jar wordcount wcinput/ wcoutput
    cat wcoutput/part-r-00000
    ```
    
     得到输出
    
    ```sh
    hadoop	1
    hdfs	1
    hello	5
    mapreduce	1
    world	1
    yarn	1
    ```

## 2.3 安装伪分布式Hadoop

### 2.3.1 修改配置文件

  - 修改  etc/hadoop/core-site.xml 文件
    ```xml
    <configuration>
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ubuntu001:9000</value>
      </property>
      
      <property>
        <name>hadoop.tmp.dir</name>
    		<value>/opt/module/hadoop/data/tmp</value>
    	</property>
    </configuration>
    ```
    
- 修改 etc/hadoop/hdfs-site.xml 文件

  ```xml
  <configuration>
        <property>
          <name>dfs.replication</name>
          <value>1</value>
        </property>
      </configuration>
  ```

- 修改 etc/hadoop/hadoop-env.xml 文件中的 JAVA_HOME，改为你机器的值

  ```sh
  echo $JAVA_HOME
  echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' > etc/hadoop/hadoop-env.xml
  ```

### 2.3.2 配置免密登陆本机

- 验证是否可以免密登陆
  ```sh
  ssh ubuntu001
  ```
- 配置免密登陆
  ```sh
  ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  chmod 0600 ~/.ssh/authorized_keys
  ```

### 2.3.3 启动Hadoop
- 格式化 namenode
  ```sh
  bin/hdfs namenode -format
  ```
- 启动 namenode和datanode服务
  ```
  sbin/start-dfs.sh
  ```
- 访问  http://ubuntu001:9870/

- 测试
  ```sh
  hdfs dfs -mkdir /user
  hdfs dfs -mkdir /user/root
  bin/hdfs dfs -mkdir input
  hdfs dfs -put etc/hadoop/*.xml input
  hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.2.jar grep input output "dfs[a-z.]+"
  bin/hdfs dfs -get output output
  cat output/*
  ```

## 2.4 单机部署时，使用Yarn

### 2.4.1 修改配置

- 修改 etc/hadoop/mapred-site.xml 文件
  ```xml
  <configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
      <name>mapreduce.application.classpath</name>    				<value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
  </configuration>
  ```
  
- 修改 etc/hadoop/yarn-site.xml 文件
  ```xml
  <configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
  </configuration>
  ```

- 启动 ResourceManager 和 nodemanager
  ```sh
  sbin/start-yarn.sh
  ```

- 访问 `http://ubuntu001:8088`
- 运行 MR 任务
- 停止 Yarn
  ```sh
  sbin/stop-yarn.sh
  ```

### 2.4.2 配置历史服务器

- 配置 mapred-site.xml 文件
  ```xml
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>ubuntu001:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>ubuntu001:19888</value>
</property>
  ```
  
- 启动历史服务器
  ```sh
  mapred --daemon start historyserver
  ```

- 查看历史服务器是否启动
  ```sh
  jps
  ```

- 查看  JobHistory: http://ubuntu001:19888/jobhistory

### 2.4.3 配置日志的聚集

- 配置 yarn-site.xml 文件
```xml
<!-- 日志聚集功能使能 --> 
<property>
  <name>yarn.log-aggregation-enable</name>
   <value>true</value>
</property>
<!-- 日志保留时间设置 7 天 --> 
<property>
  <name>yarn.log-aggregation.retain-seconds</name>
   <value>604800</value>
</property>
```

- 重启集群
```sh
mapred --daemon stop historyserver
stop-yarn.sh
stop-dfs.sh

start-dfs.sh
start-yarn.sh
mapred --daemon start historyserver
```

- 执行job
```
bin/hdfs dfs -rm -r output
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.2.jar grep input output 'dfs[a-z.]+'
```

## 2.5 完全分布式集群安装

### 2.5.1 集群规划

| 机器 | ubuntu001          | ubuntu002                    | ubuntu003                   |
| ---- | ------------------ | ---------------------------- | --------------------------- |
| HDFS | NameNode, DataNode | DataNode                     | SecondaryNameNode, DataNode |
| YARN | NodeManager        | ResourceManager, NodeManager | NodeManager                 |

### 2.5.2 配置 `hdfs-site.xml`文件

1. 修改副本数为3
2. 修改 `HDFS Secondary NameNode` 运行机器为 `ubuntu003:50090`

```xml
<property>
  <name>dfs.replication</name>
  <value>3</value>
</property>
<property>
  <name>dfs.namenode.secondary.http-address</name>
  <value>ubuntu003:50090</value>
</property>
```

### 2.5.3 配置 `yarn-site.xml` 文件

1. 添加 `ResourceManager` 运行机器为 `ubuntu002`

   ```xml
   <property>
     <name>yarn.resourcemanager.hostname</name>
     <value>ubuntu002</value>
   </property>
   ```

### 2.5.4 配置 `mapred-env.sh` 文件

1. 添加 `JAVA_HOME` 变量值

   ```sh
   export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
   ```

###  2.5.5 配置 `workers` 文件

1. 添加集群机器列表

```sh
ubuntu001
ubuntu002
ubuntu003
```

### 2.5.5  启动集群

```sh
start-dfs.sh
```

### 2.5.6 查看集群进程

1. 安装 `ansible`

   ```sh
   apt install ansible
   ```

2. 配置 `/etc/ansible/hosts` 文件, 添加如下内容

   ```sh
   # servers
   ubuntu001
   ubuntu002
   ubuntu003
   ```

3. 配置 `/etc/ansible/group_vars/servers` 文件，添加如下内容

   ```yaml
   ---
   ansible_ssh_user: root
   ```

4. 执行命令查看进程

   ```sh
   ansible -m shell -a 'jps' all
   ```

   输出

   ```sh
   ubuntu002 | CHANGED | rc=0 >>
   6354 DataNode
   7044 Jps
   ubuntu003 | CHANGED | rc=0 >>
   7164 DataNode
   7996 Jps
   7327 SecondaryNameNode
   ubuntu001 | CHANGED | rc=0 >>
   17681 NameNode
   17878 DataNode
   20491 Jps
   ```

   

5. 启动 `yarn` 服务，需要在 `ubuntu002` 上启动，即 `ResouceManager` 运行的集群上

   ```sh
   cd /opt/module/hadoop && sbin/start-yarn.sh 
   ```

6. 查看0集群运行进程(在 `ubuntu001` 上，因为 `ansible` 只在该机器上安装)

   ```sh
   ansible -m shell -a 'jps' all
   ```

   输出

   ```sh
   ubuntu003 | CHANGED | rc=0 >>
   11664 NodeManager
   7164 DataNode
   11917 Jps
   7327 SecondaryNameNode
   ubuntu002 | CHANGED | rc=0 >>
   6354 DataNode
   13652 NodeManager
   14119 Jps
   13466 ResourceManager
   ubuntu001 | CHANGED | rc=0 >>
   17681 NameNode
   17878 DataNode
   28326 Jps
   27981 NodeManager
   ```

7. 启动 `HistoryServer` 

   ```sh
   mapred --daemon start historyserver
   ```

8. 服务端口

   ```sh
   # NameNode
   http://ubuntu001:9870
   # Resource Manager
   http://ubuntu002:8088
   # JobHistory Server
   http://ubuntu001:19888
   ```

   [^https://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-common/SingleCluster.html]: Hadoop: Setting up a Single Node Cluster.
   [^https://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-common/ClusterSetup.html]: Hadoop Cluster Setup

   