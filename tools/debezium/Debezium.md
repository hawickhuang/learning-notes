# Debezium

## 概述

### 简介

- Debezium是一个将多种数据源实时变更数据捕获，形成数据流输出的开源工具

### 特性

- 可监控所有数据变化
- 低延迟生成变化事件
- 不需要更高数据模型
- 可监控删除
- 可监控元数据如事务ID
- 支持监控快照，数据过滤和变换
- 支持topic路由

### 支持数据源

- MySQL
- MongoDB
- PostgreSQL
- Oracle
- SQL Server
- DB2
- Cassandra
- Vitess

### 数据输出

- Kafka

## 架构

### 架构图

![](./debezium-architecture.png)

## 快速使用

- zk环境准备
- kafka环境准备
- mysql环境准备
- debezium/connect环境准备
  - 启动
  - 检查kafka connect状态
  - 注册connector，检测mysql数据变化
  - 测试数据同步

## 配置

### 序列化

- avro

### 信号

- Subtsignal.data.collection

### mysql配置

- binlog

  ```toml
  [mysqld]
  server-id=1
  log_bin=mysql-bin
  binlog_format=row
  binlog_row_image=full
  ```

- 用户

  ```sql
  CREATE USER 'debezium'@'%' IDENTIFIED BY 'Ku1LsgVs5XLN';
  GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'debezium' IDENTIFIED BY 'Ku1LsgVs5XLN';
  flush privileges;
  ```

### debezium配置

- name
- connector
- 数据源
- database/table 白名单
- kafka

	- host
	- topic
	- factor
	- partitions

### 连接器

```json
{
"name": "mysql-connector-db01",
"config": {
	"name": "mysql-connector-db01",
	"connector.class": "io.debezium.connector.mysql.MySqlConnector",
	"database.server.id": "1",
	"tasks.max": "3",
	"database.history.kafka.bootstrap.servers":    "172.31.47.152:9092,172.31.38.158:9092,172.31.46.207:9092",
	"database.history.kafka.topic": "schema-changes.mysql",
	"database.server.name": "mysql-db01",
	"database.hostname": "172.31.84.129",
	"database.port": "3306",
	"database.user": "bhuvi",
	"database.password": "my_stong_password",
	"database.whitelist": "proddb,test",
	"internal.key.converter.schemas.enable": "false",
	"key.converter.schemas.enable": "false",
	"internal.key.converter": "org.apache.kafka.connect.json.JsonConverter",
	"internal.value.converter.schemas.enable": "false",
	"value.converter.schemas.enable": "false",
	"internal.value.converter": "org.apache.kafka.connect.json.JsonConverter",
	"value.converter": "org.apache.kafka.connect.json.JsonConverter",
	"key.converter": "org.apache.kafka.connect.json.JsonConverter",
	"transforms": "unwrap",
	"transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState"
  "transforms.unwrap.add.source.fields": "ts_ms",
	}
}
```

###  路由配置

- topic路由
- 

