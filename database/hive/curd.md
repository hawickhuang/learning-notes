# Hive 操作

## 1. Hive 数据类型

### 1.1 基本数据类型

| Hive数据类型 | Java 数据类型 | 长度                                                     | 例子                                                                                                                       |
| ------------ | ------------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| TINYINT      | byte          | 1byte 有符号整数                                         | `-128` to `127`                                                                                                            |
| SMALLINT     | short         | 2byte 有符号整数                                         | `-32,768` to `32,767`                                                                                                      |
| INT/INTEGER  | int           | 4byte 有符号整数                                         | -2,147,483,648 to 2,147,483,647                                                                                            |
| BIGINT       | long          | 8byte 有符号整数                                         | -9,223,372,036,854,775,808` to `9,223,372,036,854,775,807                                                                  |
| BOOLEAN      | boolean       | 布尔类型，true/false                                     | TRUE/FALSE                                                                                                                 |
| FLOAT        | float         | 单精度浮点型                                             | 3.14159                                                                                                                    |
| DOUBLE       | double        | 双精度浮点型                                             | 3.14159                                                                                                                    |
| STRING       | string        | 字符序列，可以指定字符集。<br />可以使用单引号或双引号   | 'hello world'<br />"hello hive"                                                                                            |
| VARCHAR      |               | 固定长度的字符数组，0-65535                              | [Varchar](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-VarcharvarcharVarchar) |
| CHAR         |               | 固定长度的字符数组，0-255                                |                                                                                                                            |
| TIMESTAMP    |               | 时间类型                                                 |                                                                                                                            |
| DATE         |               | 日期类型                                                 |                                                                                                                            |
| INTERVAL     |               | 时间间隔单位：<br />SECOND / MINUTE / DAY / MONTH / YEAR | [Intervals](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-Intervals)           |
| BINARY       |               | 字节数组                                                 |                                                                                                                            |

### 1.2 集合数据类型

| 数据类型 | 描述   | 例子                                                    |
| -------- | ------ | ------------------------------------------------------- |
| ARRAYS   | 数组   | ARRAY<data_type>                                        |
| MAPS     | 映射   | MAP<primitive_type, data_type>                          |
| STRUCTS  | 结构体 | STRUCT<col_name : data_type [COMMENT col_comment], ...> |
| UNION    | 联合体 | UNIONTYPE<data_type, data_type, ... >                   |

### 1.3 类型转换

#### 1.3.1 隐式转换

|              | void  | boolean | tinyint | smallint | int   | bigint | float | double | decimal | string | varchar | timestamp | date  | binary |
| :----------- | :---- | :------ | :------ | :------- | :---- | :----- | :---- | :----- | :------ | :----- | :------ | :-------- | :---- | :----- |
| void to      | true  | true    | true    | true     | true  | true   | true  | true   | true    | true   | true    | true      | true  | true   |
| varchar to   | false | false   | false   | false    | false | false  | false | true   | true    | true   | true    | false     | false | false  |
| tinyint to   | false | false   | true    | true     | true  | true   | true  | true   | true    | true   | true    | false     | false | false  |
| timestamp to | false | false   | false   | false    | false | false  | false | false  | false   | true   | true    | true      | false | false  |
| string to    | false | false   | false   | false    | false | false  | false | true   | true    | true   | true    | false     | false | false  |
| smallint to  | false | false   | false   | true     | true  | true   | true  | true   | true    | true   | true    | false     | false | false  |
| int to       | false | false   | false   | false    | true  | true   | true  | true   | true    | true   | true    | false     | false | false  |
| float to     | false | false   | false   | false    | false | false  | true  | true   | true    | true   | true    | false     | false | false  |
| double to    | false | false   | false   | false    | false | false  | false | true   | true    | true   | true    | false     | false | false  |
| decimal to   | false | false   | false   | false    | false | false  | false | false  | true    | true   | true    | false     | false | false  |
| date to      | false | false   | false   | false    | false | false  | false | false  | false   | true   | true    | false     | true  | false  |
| boolean to   | false | true    | false   | false    | false | false  | false | false  | false   | false  | false   | false     | false | false  |
| binary to    | false | false   | false   | false    | false | false  | false | false  | false   | false  | false   | false     | false | true   |
| bigint to    | false | false   | false   | false    | false | true   | true  | true   | true    | true   | true    | false     | false | false  |

#### 1.3.2  强制转换

可以使用` CAST` 操作显示进行数据类型转换。例如` CAST('1' AS INT)`将把字符串`'1'` 转换成整数 `1`;如果强制类型转换失败，如执行`CAST('X' AS INT)`，表达式返回空值 `NULL`

## 2. DDL

### 2.1 数据库管理

####  2.1.1 创建数据库

```sql
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
  [COMMENT database_comment] -- 数据库描述信息
  [LOCATION hdfs_path] -- 在 HDFS 上的存放位置
  [WITH DBPROPERTIES (property_name=property_value, ...)]; --  数据库的k-v属性值，在ALTER时修改
```

- 创建一个数据库，数据的存储路径在 `HDFS`上 `/user/hive/warehouse/*.db`,加上 `if not exists`避免在数据库存在时报错

  ```sql
  create database if not exists hive_test;
  ```

  ```sh
  hdfs dfs -ls /user/hive/warehouse
  Found 2 items
  drwxr-xr-x   - hawick supergroup          0 2021-04-10 10:10 /user/hive/warehouse/hive_test.db
  drwxr-xr-x   - hawick supergroup          0 2021-04-10 08:49 /user/hive/warehouse/test
  ```

#### 2.1.2 查看数据库

```sql
hive (default)> show databases; -- 显示所有数据库列表
OK
database_name
default
hive_test
Time taken: 0.195 seconds, Fetched: 2 row(s)

hive (default)> desc database hive_test; -- 显示数据库详情信息
OK
db_name	comment	location	owner_name	owner_type	parameters
hive_test		hdfs://ubuntu001:9000/user/hive/warehouse/hive_test.db	hawick	USER
Time taken: 0.033 seconds, Fetched: 1 row(s)

hive (default)> use hive_test; -- 切换当前数据库
OK
Time taken: 0.039 seconds
```



#### 2.1.3 修改数据库

```sql
ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);   -- (Note: SCHEMA added in Hive 0.14.0) 修改属性信息
ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;   -- (Note: Hive 0.13.0 and later; SCHEMA added in Hive 0.14.0) 修改所属用户信息
ALTER (DATABASE|SCHEMA) database_name SET LOCATION hdfs_path; -- (Note: Hive 2.2.1, 2.4.0 and later) 修改存储路径
```

只可以修改 `属性信息`、`所属用户信息`和 `存储路径`，库名不可以修改

```sql
 alter database hive_test set dbproperties('createtime'='20210408');
 alter database hive_test set owner user hive;
 alter database hive_test set location 'hdfs://ubuntu001:9000/user/hive/warehouse-test/'; -- 需要先创建目录，且使用hdfs完整URI hdfs dfs -mkdir /user/hive/warehouse-test
 
```

#### 2.1.4 删除数据库

```sql
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

```sql
drop database hive_test; -- HDFS上最后关联的目录会被删除
```

### 2.2 表管理

#### 2.2.1 创建表

```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- EXTERNAL指定创建外部表，不指定是内部表(managed table);
  [(col_name data_type [column_constraint_specification] [COMMENT col_comment], ... [constraint_specification])] -- 字段和字段约束定义
  [COMMENT table_comment] -- 表注释
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] -- 分区信息
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] -- 分桶，排序
  [SKEWED BY (col_name, col_name, ...)                  -- 倾斜列，会对经常出现的列进行切分或按目录存储
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] -- 行格式
   [STORED AS file_format] -- 存储的文件类型
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path] -- 表存储路径
  [TBLPROPERTIES (property_name=property_value, ...)]   -- 表属性信息
  [AS select_statement];   -- 依据子查询创建，不支持外部表
 
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name -- 依照已存在的表创建新表
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];
 
data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type  -- (Note: Available in Hive 0.7.0 and later)
 
primitive_type
  : TINYINT
  | SMALLINT
  | INT
  | BIGINT
  | BOOLEAN
  | FLOAT
  | DOUBLE
  | DOUBLE PRECISION -- (Note: Available in Hive 2.2.0 and later)
  | STRING
  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
  | DATE        -- (Note: Available in Hive 0.12.0 and later)
  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
  | CHAR        -- (Note: Available in Hive 0.13.0 and later)
 
array_type
  : ARRAY < data_type >
 
map_type
  : MAP < primitive_type, data_type >
 
struct_type
  : STRUCT < col_name : data_type [COMMENT col_comment], ...>
 
union_type
   : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)
 
row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char]   -- (Note: Available in Hive 0.13 and later)
  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
 
file_format:
  : SEQUENCEFILE -- 存储为压缩的序列化文件
  | TEXTFILE    -- Default, 取决于 hive.default.fileformat 配置
  | RCFILE      -- RCFILE（列式存储）
  | ORC         -- ORC文件格式（列式存储）
  | PARQUET     -- Parquet文件格式（列式存储）
  | AVRO        -- Avro格式
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname -- 指定 InputFormat 和 OutputFormat 类名
 
column_constraint_specification: -- 列约束
  : [ PRIMARY KEY|UNIQUE|NOT NULL|DEFAULT [default_value]|CHECK  [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]
 
default_value:
  : [ LITERAL|CURRENT_USER()|CURRENT_DATE()|CURRENT_TIMESTAMP()|NULL ] 
 
constraint_specification: -- 表约束信息
  : [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, PRIMARY KEY (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, CONSTRAINT constraint_name FOREIGN KEY (col_name, ...) REFERENCES table_name(col_name, ...) DISABLE NOVALIDATE 
    [, CONSTRAINT constraint_name UNIQUE (col_name, ...) DISABLE NOVALIDATE RELY/NORELY ]
    [, CONSTRAINT constraint_name CHECK [check_expression] ENABLE|DISABLE NOVALIDATE RELY/NORELY ]
```

- 外部表与内部表的区别
  - 只有内部表支持 `ARCHIVE/UNARCHIVE/TRUNCATE/MERGE/CONCATENATE`;
  - `DROP`会删除内部表数据，而外部表只会删除元数据；
  - 只有内部表支持 `ACID` 和 `事务`；
  - 只有内部表支持 `查询结果缓存`；
  - 只有外部表支持 `RELY` 约束；
  - 某些 `物化视图` 只有外部表支持； 

### 2.2.2 修改表

```sql
-- 表基本信息修改
ALTER TABLE table_name RENAME TO new_table_name; -- 重命名
ALTER TABLE table_name SET TBLPROPERTIES table_properties; -- 修改表属性
table_properties:
  : (property_name = property_value, property_name = property_value, ... )

-- 表的 SerDe 修改
ALTER TABLE table_name [PARTITION partition_spec] SET SERDE serde_class_name [WITH SERDEPROPERTIES serde_properties]; -- 修改表的解析方法
ALTER TABLE table_name [PARTITION partition_spec] SET SERDEPROPERTIES serde_properties; -- 给表添加解析方法属性
serde_properties:
  : (property_name = property_value, property_name = property_value, ... )
ALTER TABLE table_name [PARTITION partition_spec] UNSET SERDEPROPERTIES (property_name, ... ); -- 删除表的某个解析方法属性
ALTER TABLE table_name CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name, ...)]
  INTO num_buckets BUCKETS; -- 给表添加分桶/排序属性，会改变物理存储逻辑

-- 表的 SKEW 修改
ALTER TABLE table_name SKEWED BY (col_name1, col_name2, ...)
  ON ([(col_name1_value, col_name2_value, ...) [, (col_name1_value, col_name2_value), ...]
  [STORED AS DIRECTORIES]; -- 给表添加倾斜属性及其存储目录分组      
ALTER TABLE table_name NOT SKEWED; -- 删除表的倾斜设置
ALTER TABLE table_name NOT STORED AS DIRECTORIES; -- 删除表的分目录存储设置
ALTER TABLE table_name SET SKEWED LOCATION (col_name1="location1" [, col_name2="location2", ...] ); -- 修改表的分目录存储时的目录

-- 表的约束修改     
ALTER TABLE table_name ADD CONSTRAINT constraint_name PRIMARY KEY (column, ...) DISABLE NOVALIDATE; -- 给表添加主键约束
ALTER TABLE table_name ADD CONSTRAINT constraint_name FOREIGN KEY (column, ...) REFERENCES table_name(column, ...) DISABLE NOVALIDATE RELY; -- 给表添加外键约束
ALTER TABLE table_name ADD CONSTRAINT constraint_name UNIQUE (column, ...) DISABLE NOVALIDATE; -- 给列添加唯一约束
ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name NOT NULL ENABLE; -- 给列添加非空约束
ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name DEFAULT default_value ENABLE; -- 修改列名及其约束
ALTER TABLE table_name CHANGE COLUMN column_name column_name data_type CONSTRAINT constraint_name CHECK check_expression ENABLE; -- 修改列名即检查表达式
ALTER TABLE table_name DROP CONSTRAINT constraint_name; -- 删除表的某个约束

-- 表的分区修改
ALTER TABLE table_name ADD [IF NOT EXISTS] PARTITION partition_spec [LOCATION 'location'][, PARTITION partition_spec [LOCATION 'location'], ...]; -- 添加分区设置 
partition_spec:
  : (partition_column = partition_col_value, partition_column = partition_col_value, ...)
ALTER TABLE table_name PARTITION partition_spec RENAME TO PARTITION partition_spec; -- 修改分区设置
ALTER TABLE table_name_2 EXCHANGE PARTITION (partition_spec) WITH TABLE table_name_1; -- 将table_name_1的分区移动到table_name_2
ALTER TABLE table_name_2 EXCHANGE PARTITION (partition_spec, partition_spec2, ...) WITH TABLE table_name_1; -- 将1分区移动到多个表
MSCK [REPAIR] TABLE table_name [ADD/DROP/SYNC PARTITIONS]; -- 元数据的分区信息添加/删除/同步（HDFS存在但元数据无/HDFS已删除但元数据存在）
ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec[, PARTITION partition_spec, ...] [PURGE];  -- 删除分区，PURGE将直接删除，不会进入回收箱
ALTER TABLE table_name ARCHIVE PARTITION partition_spec; -- 归档后分区
ALTER TABLE table_name UNARCHIVE PARTITION partition_spec; -- 不归档分区
       
-- 表或分区的修改
ALTER TABLE table_name [PARTITION partition_spec] SET FILEFORMAT file_format; -- 修改表或分区的文件格式
ALTER TABLE table_name [PARTITION partition_spec] SET LOCATION "new location"; -- 修改表或分区的存储路径
ALTER TABLE table_name TOUCH [PARTITION partition_spec]; -- 用于触发勾子
ALTER TABLE table_name [PARTITION partition_spec] ENABLE|DISABLE NO_DROP [CASCADE]; -- 开启/关闭保护 
ALTER TABLE table_name [PARTITION partition_spec] ENABLE|DISABLE OFFLINE; -- OFFLINE开启后，元数据可以访问，但表/分区不可查询
ALTER TABLE table_name [PARTITION (partition_key = 'partition_value' [, ...])]
  COMPACT 'compaction_type'[AND WAIT]
  [WITH OVERWRITE TBLPROPERTIES ("property"="value" [, ...])]; -- 启动压缩，默认不需要手动压缩，Hive会自动压缩
ALTER TABLE table_name [PARTITION (partition_key = 'partition_value' [, ...])] CONCATENATE; -- 合并文件，当有很多小文件时
ALTER TABLE table_name [PARTITION (partition_key = 'partition_value' [, ...])] UPDATE COLUMNS; -- 同步表结构
       
-- 列信息修改
ALTER TABLE table_name [PARTITION partition_spec] CHANGE [COLUMN] col_old_name col_new_name column_type
  [COMMENT col_comment] [FIRST|AFTER column_name] [CASCADE|RESTRICT]; -- 列的基本信息修改
ALTER TABLE table_name [PARTITION partition_spec]  ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) [CASCADE|RESTRICT]; -- 添加或替换列（替换列会删除已有的所有列，再创建新的列，只支持Hive自由的Serde）
```

### 2.2.3 删除表/截断表

```sql
DROP TABLE [IF EXISTS] table_name [PURGE]; -- 删除表
TRUNCATE [TABLE] table_name [PARTITION partition_spec]; -- 清空表或分区
partition_spec:
  : (partition_column = partition_col_value, partition_column = partition_col_value, ...)
```

## 2.3 视图管理

```sql
-- 创建视图
CREATE VIEW [IF NOT EXISTS] [db_name.]view_name [(column_name [COMMENT column_comment], ...) ]
  [COMMENT view_comment]
  [TBLPROPERTIES (property_name = property_value, ...)]
  AS SELECT ...;
-- 修改视图
ALTER VIEW [db_name.]view_name SET TBLPROPERTIES table_properties; 
table_properties:
  : (property_name = property_value, property_name = property_value, ...)
ALTER VIEW [db_name.]view_name AS select_statement; -- 从子查询中创建视图
--删除视图
DROP VIEW [IF EXISTS] [db_name.]view_name;
-- 创建实例化视图
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [db_name.]materialized_view_name
  [DISABLE REWRITE]
  [COMMENT materialized_view_comment]
  [PARTITIONED ON (col_name, ...)]
  [CLUSTERED ON (col_name, ...) | DISTRIBUTED ON (col_name, ...) SORTED ON (col_name, ...)]
  [
    [ROW FORMAT row_format]
    [STORED AS file_format]
      | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]
AS SELECT ...;
-- 删除实例化视图
DROP MATERIALIZED VIEW [db_name.]materialized_view_name;
-- 修改实例化视图
ALTER MATERIALIZED VIEW [db_name.]materialized_view_name ENABLE|DISABLE REWRITE;
```

### 2.4 宏管理

```sql
-- 创建临时宏
CREATE TEMPORARY MACRO macro_name([col_name col_type, ...]) expression;
-- 删除临时宏
DROP TEMPORARY MACRO [IF EXISTS] macro_name;
```

### 2.5 函数管理

```sql
-- 创建临时函数
CREATE TEMPORARY FUNCTION function_name AS class_name;
-- 删除临时函数
DROP TEMPORARY FUNCTION [IF EXISTS] function_name
-- 创建永久函数
CREATE FUNCTION [db_name.]function_name AS class_name
  [USING JAR|FILE|ARCHIVE 'file_uri' [, JAR|FILE|ARCHIVE 'file_uri'] ];
-- 删除永久函数
DROP FUNCTION [IF EXISTS] function_name;
```

### 2.6  Show & Desc

```sql
SHOW (DATABASES|SCHEMAS) [LIKE 'identifier_with_wildcards']; -- 列出所有数据仓库
SHOW TABLES [IN database_name] ['identifier_with_wildcards']; -- 列出所有表
SHOW VIEWS [IN/FROM database_name] [LIKE 'pattern_with_wildcards']; -- 列出所有视图
SHOW MATERIALIZED VIEWS [IN/FROM database_name] [LIKE 'pattern_with_wildcards']; -- 列出所有实例化视图
SHOW PARTITIONS table_name; -- 列出表的分区
SHOW TABLE EXTENDED [IN|FROM database_name] LIKE 'identifier_with_wildcards' [PARTITION(partition_spec)]; -- 列出表或分区的扩展信息
SHOW TBLPROPERTIES tblname; -- 列出表的所有属性信息
SHOW TBLPROPERTIES tblname("foo"); -- 只展示表的foo属性值
SHOW CREATE TABLE ([db_name.]table_name|view_name); -- 列出表/视图的创建语句
SHOW COLUMNS (FROM|IN) table_name [(FROM|IN) db_name]; -- 列出表的所有列
SHOW FUNCTIONS [LIKE "<pattern>"]; -- 列出函数
-- 列出锁信息
SHOW LOCKS <table_name>;
SHOW LOCKS <table_name> EXTENDED;
SHOW LOCKS <table_name> PARTITION (<partition_spec>);
SHOW LOCKS <table_name> PARTITION (<partition_spec>) EXTENDED;
SHOW LOCKS (DATABASE|SCHEMA) database_name;  

SHOW CONF <configuration_name>; -- 展示配置信息
SHOW TRANSACTIONS; -- 展示事务信息
SHOW COMPACTIONS; -- 展示压缩信息

DESCRIBE DATABASE [EXTENDED] db_name; -- 数据仓库的详细描述信息
DESCRIBE SCHEMA [EXTENDED] db_name; -- 同上一句
DESCRIBE [EXTENDED | FORMATTED]
    [db_name.]table_name [PARTITION partition_spec] [col_name ( [.field_name] | [.'$elem$'] | [.'$key$'] | [.'$value$'] )* ]; -- 表或分区或字段的详细描述信息
```

### 2.7 实践

- 创建普通表( 内部表)

  ```sql
  create table if not exists student(
      id int,
      name string
  ) row format delimited fields terminated by '\t'
  stored as textfile
  location '/user/hive/warehouse/student';
  ```

- 从已存在的表创建新表

  ```sql
  create table if not exists student_like like student;
  ```

- 从子查询中创建新表

  ```sql
  create table if not exists student2 as select id, name from student;
  ```

- 创建外部表

  ```sh
  # 现在 HDFS 上导入测试数据文件（hive安装包examples目录下有很多测试文件）
  hdfs dfs -mkdir /user/hive/warehouse/student_ex
  hdfs dfs -put examples/files/students.txt /user/hive/warehouse/student_ex
  ```

  ```sql
  create external table student_ex(
      name string,
      col2 int, -- 很多字段不知道是啥，随便用吧
      col3 float
  ) row format delimited fields terminated by '\t'
  location '/user/hive/warehouse/student_ex';
  select * from student_ex; --  查看数据
  desc formatted student_ex; -- 查看表信息
  drop table student_ex; -- 删除表，再去 HDFS 上查看可知数组文件仍然存在
  ```

- 内部表与外部表的互相转换

  ```sql
  alter table student set tblproperties('external'='true');
  lter table student set tblproperties('external'='false');
  ```

- 分区表，即在 `HDFS`上分目录存储。将大数据集根据业务需求分割成小的数据集，从而提高 `WHERE`查询的效率

  ```sql
  -- 创建分区表
  create table partition_test(
      id int,
      col2 string,
      col3 string
  ) partitioned by (month string) -- 分区字段不能是已有字段
  row format delimited fields terminated by '\t';
  -- hdfs dfs -ls /user/hive/warehouse 查看 HDFS 目录
  
  -- 加载数据，数据自己随便搞吧
  load data local inpath '/opt/module/hive/partition_test_202102.txt' into table partition_test partition(month='202102');
  load data local inpath '/opt/module/hive/partition_test_202103.txt' into table partition_test partition(month='202103');
  load data local inpath '/opt/module/hive/partition_test_202104.txt' into table partition_test partition(month='202104');
  -- hdfs dfs -ls /user/hive/warehouse/partition_test 查看 HDFS 目录
  
  -- 查询单个分区或者多个分区的数据
  select * from partition_test where month='202102';
  select * from partition_test where month='202102' 
      union
       select * from partition_test where month='202103'
      union
       select * from partition_test where month='202104';
       
  -- 添加分区
  alter table partition_test add partition(month='202105');
  alter table partition_test add partition(month='202106') partition(month='202107');
  
  -- 查看分区信息
  show partitions partition_test;
  desc formatted partition_test
  
  -- 分区同步，1）先在 HDFS 上创建数据，再使用 MSCK 修复；2）在 HDFS 上创建数据，再添加分区；3）在 HDFS 上创建目录，load数据到分区
  -- hdfs dfs -mkdir /user/hive/warehouse/partition_test/month=202105 
  -- hdfs dfs -put partition_test_202105.txt /user/hive/warehouse/partition_test/month=202105
  -- hdfs dfs -mkdir /user/hive/warehouse/partition_test/month=202106
  -- hdfs dfs -put partition_test_202106.txt /user/hive/warehouse/partition_test/month=202106
  msck repair table partition_test;
  select * from partition_test where month='202106';
  show partitions partition_test;
  
  -- 二级分区
  -- 创建表
  create table partition_test2(
      id int,
      col1 string,
      co12 string
  ) partitioned by (month string, day string)
  row format delimited fields terminated by '\t';
  -- 加载数据
  load data local inpath '/opt/module/hive/partition_test_202102.txt' into table partition_test2 partition(month='202102',day='11');
  load data local inpath '/opt/module/hive/partition_test_202102.txt' into table partition_test2 partition(month='202102',day='21');
  load data local inpath '/opt/module/hive/partition_test_202103.txt' into table partition_test2 partition(month='202103', day='11');
  load data local inpath '/opt/module/hive/partition_test_202103.txt' into table partition_test2 partition(month='202103', day='21');
  -- 查询数据
  select * from partition_test2 where month='202102' and day='11';
  select * from partition_test2 where month='202102' and day='11'
      union
      select * from partition_test2 where month='202102' and day='21';
  ```

- 修改表

  ```sql
  alter table student2 rename to student3; -- 修改表名
  alter table student3 add columns(age int); -- 添加列
  alter table student3 change column age score int; -- 修改列
  alter table student3 replace columns(no int, nickname string, gender string); -- 替换列
  desc student3; -- 查询表结构，每次执行完都可以查看一下
  drop table student3; -- 删除表
  ```

## 3. DML

### 3.1 数据加载

```sql
-- 从本地文件或URI中加载数据，前文已操作过
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]; 
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [INPUTFORMAT 'inputformat' SERDE 'serde'];

-- 导入导出
EXPORT TABLE tablename [PARTITION (part_column="value"[, ...])]
  TO 'export_target_path' [ FOR replication('eventid') ];
IMPORT [[EXTERNAL] TABLE new_or_original_tablename [PARTITION (part_column="value"[, ...])]]
  FROM 'source_path'
  [LOCATION 'import_target_path'];

-- 从查询结果中创建数据
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1 FROM from_statement; -- OVERWRITE 会覆盖原有数据； PARTITION 会动态重写分区
INSERT INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement; -- 追加形式
-- 插入多表
FROM from_statement
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1
[INSERT OVERWRITE TABLE tablename2 [PARTITION ... [IF NOT EXISTS]] select_statement2]
[INSERT INTO TABLE tablename2 [PARTITION ...] select_statement2] ...;
FROM from_statement
INSERT INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1
[INSERT INTO TABLE tablename2 [PARTITION ...] select_statement2]
[INSERT OVERWRITE TABLE tablename2 [PARTITION ... [IF NOT EXISTS]] select_statement2] ...;

-- 将查询结果写入文件系统
INSERT OVERWRITE [LOCAL] DIRECTORY directory1
  [ROW FORMAT row_format] [STORED AS file_format] 
  SELECT ... FROM ...
-- 写入多个文件系统
FROM from_statement
INSERT OVERWRITE [LOCAL] DIRECTORY directory1 select_statement1
[INSERT OVERWRITE [LOCAL] DIRECTORY directory2 select_statement2] ...

-- 将 VALUES 插入表
INSERT INTO TABLE tablename [PARTITION (partcol1[=val1], partcol2[=val2] ...)] VALUES values_row [, values_row ...]
```

### 3.2 数据修改/删除/合并

```sql
UPDATE tablename SET column = value [, column = value ...] [WHERE expression]; -- 修改，一般不用，请回忆HDFS使用场景
DELETE FROM tablename [WHERE expression]; -- 删除

-- 合并
MERGE INTO <target table> AS T USING <source expression/table> AS S
ON <boolean expression1>
WHEN MATCHED [AND <boolean expression2>] THEN UPDATE SET <set clause list>
WHEN MATCHED [AND <boolean expression3>] THEN DELETE
WHEN NOT MATCHED [AND <boolean expression4>] THEN INSERT VALUES<value list>
```

### 3.3 实践

- 从文件中加载

  ```sql
  load data local inpath '/opt/module/hive/students.txt' into table student; -- 本地文件导入
  -- dfs -mkdir /user/hawick/hive; 
  -- dfs -put /opt/module/hive/students.txt /user/hawick/hive;
  load data inpath '/user/hawick/hive/students.txt' into table student; -- 从 HDFS 上导入
  load data inpath '/user/hawick/hive/students.txt' overwrite into table student; -- 覆盖原有数据
  
  -- insert 插入，先执行 drop table student；insert不支持插入部分字段
  create table student(
      id int, name string
  ) partitioned by (month string) 
  row format delimited fields terminated by '\t';
  insert into table student partition(month="202104") values(1,'hello'),(2,'world'); -- 插入数据
  insert into table student partition(month='202105')
      select id,name from student where month='202104'; -- 通过查询结果插入数据
  from student -- 通过查询结果，多表/多分区插入（overwrite覆盖数据）
      insert overwrite table student partition(month='202106')
      select id,name where month='202104'
      insert overwrite table student partition(month='202107')
      select id,name where month='202105';
      
  -- location
  create table if not exists student2 as select id,name from student; -- 根据查询结果创建表并将结果插入
  create external table if not exists student3(
      id int, name string
  ) row format delimited fields terminated by '\t'
  location '/user/hive/warehouse/student_lo'; -- 创建表时指定 location，会自动加载目录下数据
  
  -- export/import，导出时带分区，则需要导入一个新表或者无该分区的表；主要用于集群间数据迁移
  export table student partition(month="202104") to '/user/hive/warehouse/export';
  import table student4  from '/user/hive/warehouse/export';
  
  -- 清空/删除
  truncate table student4;
  drop table student4;
  ```

## 4. 查询

### 4.1 查询语句

```sql
[WITH CommonTableExpression (, CommonTableExpression)*]  
SELECT [ALL | DISTINCT] select_expr, select_expr, ... -- DISTINCT 去除重复值的行
  FROM table_reference
  [WHERE where_condition] -- 条件查询
  [GROUP BY col_list] -- 分组，通常与聚合函数配合使用
  [ORDER BY col_list] -- 全局排序
  [CLUSTER BY col_list -- 当 DISTRIBUTE BY 与SORT BY 作用于同一个字段时，同义，但 CLUSTER BY不能设定排序方式
    | [DISTRIBUTE BY col_list] [SORT BY col_list] -- 分区排序
  ]
 [LIMIT [offset,] rows] -- limit，设置返回行数和偏移
 
 -- 分组
groupByClause: GROUP BY groupByExpression (, groupByExpression)*
groupByExpression: expression
groupByQuery: SELECT expression (, expression)* FROM src groupByClause?
-- 全局排序
colOrder: ( ASC | DESC )
colNullOrder: (NULLS FIRST | NULLS LAST)        
orderBy: ORDER BY colName colOrder? colNullOrder? (',' colName colOrder? colNullOrder?)*
query: SELECT expression (',' expression)* FROM src orderBy
-- 分区排序
colOrder: ( ASC | DESC )
sortBy: SORT BY colName colOrder? (',' colName colOrder?)*
query: SELECT expression (',' expression)* FROM src sortBy
-- JOIN语句
join_table:
    table_reference [INNER] JOIN table_factor [join_condition]
  | table_reference {LEFT|RIGHT|FULL} [OUTER] JOIN table_reference join_condition
  | table_reference LEFT SEMI JOIN table_reference join_condition
  | table_reference CROSS JOIN table_reference [join_condition] 
 
table_reference:
    table_factor
  | join_table
 
table_factor:
    tbl_name [alias]
  | table_subquery alias
  | ( table_references )
 
join_condition:
    ON expression
-- UNION 语句
select_statement UNION [ALL | DISTINCT] select_statement UNION [ALL | DISTINCT] select_statement ...
-- 采样桶
table_sample: TABLESAMPLE (BUCKET x OUT OF y [ON colname])
-- 公用表
withClause: cteClause (, cteClause)*
cteClause: cte_name AS (select statment)
```

### 4.2 实践

- 基本查询， `Hive`中支持很多操作符和内建了很多自定义函数，可通过链接查看 [操作符与自定义函数](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)

  ```sql
  -- 数据准备
  create table if not exists dept(
      id int,
      dept_name string    
  ) row format delimited fields terminated by '|';
  create table if not exists emp(
      name string,
      dept_id int,
      loc_id int
  ) row format delimited fields terminated by '|';
  create table if not exists loc(
      name string,
      loc_id int,
      state_code string,
      city_code string
  ) row format delimited fields terminated by '|';
  create table if not exists emp2(
      id int,
      first_name string,
      last_name string,
      leader_id int,
      birthday date,
      birthtime timestamp,
      salary int,
      reward float,
      board float,
      dept_id int
  ) row format delimited fields terminated by '|';
  load data local inpath '/opt/module/hive/examples/files/dept.txt' into table dept;
  load data local inpath '/opt/module/hive/examples/files/emp.txt' into table emp;
  load data local inpath '/opt/module/hive/examples/files/loc.txt' into table loc;
  load data local inpath '/opt/module/hive/examples/files/emp2.txt' into table emp2;
  
  -- 查询，表名/列名 不区分大小写
  select * from dept; -- 全表查询
  select id, dept_name from dept; -- 查询特定字段
  select id, dept_name as name from dept; -- 列别名
  select id+10  from dept; -- 算术运算
  select count(*) from dept; -- 内建函数
  select * from dept limit 3,2; -- 按0计数，从第3行开始，返回2行
  select * from dept where id >34; -- where 条件语句，可配合分区使用；支持的逻辑&比较操作符可查看上面给出的文档链接
  select count(*) from emp group by dept_id; -- 聚合
  select dept_id, avg(salary) avg_sal from emp2 group by dept_id having avg_sal > 2000; -- 查询平均工资大于2000的部门
  from emp -- 高级聚合
  insert overwrite table dept_count
      select emp.dept_id, count(*) group by emp.dept_id -- 各部门员工数
  insert overwrite directory '/user/hawick/emp_loc_count'
      select emp.loc_id, count(*) group by emp.loc_id; -- 各个地方员工数
  ```

- JOIN，Hive只支持等值连接

  ```sql
  -- 内连接(inner join和join)，进行连接的两个表中都存在，且条件相匹配的数据被保留；下面语句不会返回dept中id为(35,36,37)的数据
  select * from dept join emp on dept.id=emp.dept_id; -- 查询员工信息及其部门信息
  select * from dept d join emp e on d.id=e.dept_id; -- 使用别名， 简化查询/提高可读性/提高执行效率
  -- 左外连接，左表中符合where子句的数据都会被返回，不管 ON 子句条件是否匹配
  select e.dept_id,e.name, d.id from emp e left join dept d on e.dept_id=d.id;
  select e.dept_id,e.name, d.id from dept d left join emp e on e.dept_id=d.id; -- 会返回(NULL	NULL	35),(NULL	NULL	36),(NULL	NULL	37)
  -- 右外连接，右表中符合WHERE子句的数据都会被返回，不管ON子句条件是否匹配
  select e.dept_id,e.name, d.id from emp e left join dept d on e.dept_id=d.id; -- -- 会返回(NULL	NULL	35),(NULL	NULL	36),(NULL	NULL	37)
  select e.dept_id,e.name, d.id from dept d left join emp e on e.dept_id=d.id;
  -- 满外连接，所有表中符合WHERE子句的数据都会被返回
  select e.dept_id,e.name, d.id from emp e full join dept d on e.dept_id=d.id;
  -- 多表连接：连接 n 个表，至少需要 n-1 个连接条件。例如:连接三个表，至少需要两个连 接条件。
  select e.name,e.dept_id,e.loc_id,d.dept_name,l.name from emp e join dept d on e.dept_id=d.id join loc l on e.loc_id=l.loc_id;
  -- 无ON子句，或连接条件无效，或所有表中的所有行互相连接 会出现 笛卡尔乘积的情况
  select emp.name,loc.name from emp,loc;
  ```

- 排序

  ```sql
  -- order by: 全局排序，只有一个 Reducer
  select * from emp2 order by salary;
  select * from emp2 order by salary,reward desc; -- 降序,多个列
  -- sort by: 全局排序对于大数据集时效率非常低，使用 sort by 在每个 MapReduce 内部排序，全局结果集并没有排序
  set mapreduce.job.reduces=3; -- 设置reduce个数，默认只会启用一个reduce
  select * from emp2 sort by dept_id desc; -- 按部门ID降序排序
  insert overwrite local directory '/opt/module/hive/sortby_emp2' select * from emp2 sort by dept_id desc; -- 将结果写入本地文件，可查看执行结果；可以看到有三个分区，各自分区有序
  -- distribute by： distribute by 的分区规则是根据分区字段的 hash 码与 reduce 的个数进行模除后，余数相同的分到一个区；
  insert overwrite local directory '/opt/module/hive/emp2_dist' select * from emp2 distribute by dept_id sort by id desc;
  -- cluster by: 当 distribute by 和 sorts by 字段相同时，可以使用 cluster by 方式
  select * from emp2 cluster by dept_id;
  ```

- 分桶与抽样

  分区提供了隔离数据和优化查询的便利方式，但分区针对的是数据的存储路径，粒度较粗；于是 `Hive` 提供了更细粒度的方式：分桶，针对数据文件。

  ```sql
  -- 需要将强制reduce的数量设回默认
  set mapreduce.job.reduces=-1;
  -- 创建分桶表
  create table bucket_test(
      id int,
      name string
  ) clustered by (id)
  into 4 buckets
  row format delimited fields terminated by '\t';
  -- 加载数据
  load data local inpath '/opt/module/hive/bucket_test.txt' into table bucket_test;
  -- 查看目录，分桶规则：对分桶字段进行哈希，然后对桶的数目取余来决定记录存放在哪个桶
  dfs -ls -R /user/hive/warehouse/bucket_test -- 注意这里的目录时分区，不是分桶
  ```

- 分桶抽样查询

  对于非常大的数据集，我们查询所有结果并不是一个好的方案；如果允许抽取一部分随机数据可以代表整体的特性，那么可以通过抽样来实现。`Hive`通过 `tablesample`语句实现了抽样的功能。

  `tablesample(bucket x out of y)`:  其中 `y` 是桶的数目的倍数或者因子；该语句表示抽取 `bucket_count / y` 个桶的数据，`x` 表示从哪个桶开始抽取。当抽取多个桶的数据时，以后分区号为 `x+ny`，如此递推。例如当 `bucket_count=4`、`y=2`、`x=1`时，表示抽取 `4/2=2`个桶的数据，从桶号(1为第一个) `x=1`开始，第二个桶为 `x+1*y=3` 。注意，x 的值必须小于等于 y 的值。

  ```sql
  select * from bucket_test tablesample(bucket 1 out of 4 on id); -- 通过x，y方式抽样
  select * from bucket_test tablesample(0.1 percent); -- 通过百分比抽样（数据量大小的百分比，不是行数）
  set hive.sample.seednumber=<INTEGER>; -- 修改随机因子，避免每次抽取的结果一样
  select * from bucket_test tablesample( 5 rows); -- 抽样返回的行数
  ```

- 常用的查询函数（更多函数请参考[Built-in Functions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-Built-inFunctions)）

  ```sql
  -- NVL(value, default_value), 如果value为NULL，则返回default_value
  select id,first_name,nvl(reward,-1111) from emp2;
  -- CASE a WHEN b THEN c [WHEN d THEN e]* [ELSE f] END, 当a=b，返回c；当a=d，返回e；否则返回f
  -- CASE WHEN a THEN b [WHEN c THEN d]* [ELSE e] END, 当a为true，返回b；当c为true，返回d；否则返回e
  select dept_id,
      sum(case dept_id when 10 then 1 else 0 end) count_10,
      sum(case dept_id when 20 then 1 else 0 end) count_20,
      sum(case dept_id when 30 then 1 else 0 end) count_30
  from emp2 group by dept_id; -- 求出每个部门有多少人，跟group by的结果一样
  ```

- 窗口函数

  - 应用场景：`分区排序`/`动态 Group By` / `Top N` / `累计计算` / `层次查询`

  - 主要函数：

    - `LEAD`: 往后 n 行数据

    - `LAG`: 往前 n 行数据

    - `FIRST_VALUE`: 想要的列的第一个值

    - `LAST_VALUE`: 想要的列的最后一个值

    - `OVER`:

      ```sql
      -- CURRENT ROW:当前行
      -- n PRECEDING:往前 n 行数据
      -- n FOLLOWING:往后 n 行数据
      -- UNBOUNDED:起点，UNBOUNDED PRECEDING 表示从前面的起点，UNBOUNDED FOLLOWING 表示到后面的终点
      (ROWS | RANGE) BETWEEN (UNBOUNDED | [num]) PRECEDING AND ([num] PRECEDING | CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
      (ROWS | RANGE) BETWEEN CURRENT ROW AND (CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
      (ROWS | RANGE) BETWEEN [num] FOLLOWING AND (UNBOUNDED | [num]) FOLLOWING
      ```

      

    - `RANK`: 生成数据项在分组中的排名，有排名相等时，会留下空位(如1/1/3)

    - `DENSE_RANK`: 生成数据项在分组中的排名，有排名相等时，不会留下空位(如1/1/2)

    - `ROW_NUMBER`: 从1开始，按照顺序，生成分组内记录的序列。

    - `CUME_DIST`: 小于等于当前值的行数/分组内总行数。

    - `PERCENT_RANK`: `(分组内当前行的RANK值-1)` / `(分组内总行数-1)`

    - `NTILE`: 用于将分组数据按照顺序切分成n片，返回当前切片值，如果切片不均匀，默认增加第一个切片的分布。

  ```sh
  # 数据准备
  cat window_test.txt
  jack,2021-01-01,10
  tony,2021-01-02,15
  jack,2021-02-03,23
  tony,2021-01-04,29
  jack,2021-01-05,46
  jack,2021-04-06,42
  tony,2021-01-07,50
  jack,2021-01-08,55
  mart,2021-04-08,62
  mart,2021-04-09,68
  neil,2021-05-10,12
  mart,2021-04-11,75
  neil,2021-06-12,80
  ```

  

  ```sql
  -- 创建表
  create table window_test(
      name string,
      orderdate string,
      cost int
  ) row format delimited fields terminated by ',';
  -- 加载数据
  load data local inpath '/opt/module/hive/window_test.txt' into table window_test;
  -- 查询4月份购买过商品的顾客及次数
  select name, count(*) from window_test 
  where substring(orderdate, 1, 7) = '2021-04' group by name;
  -- 查询每个顾客的购买明细及其购买总额
  select name, orderdate, cost, sum(cost) over (partition by month(orderdate)) from window_test;
  -- 所有行相加
  select name, orderdate, cost, sum(cost) over() as sample1 from window_test;
  -- 按 name 分组，组内数据累加
  select name, orderdate, cost, sum(cost) over(partition by name order by orderdate) as sample1 from window_test;
  -- 由起点到当前行的聚合
  select name, orderdate, cost, sum(cost) over(partition by name order by orderdate rows between unbounded preceding and current row) as sample1 from window_test;
  -- 当前行和前面一行、后面一行的聚合
  select name, orderdate, cost, sum(cost) over(partition by name order by orderdate rows between 1 preceding and 1 following) as sample1 from window_test;
  -- 当前行及后面所有行的聚合
  select name, orderdate, cost, sum(cost) over(partition by name order by orderdate rows between current row and unbounded following) as sample1 from window_test;
  -- 顾客上一次购买的时间
  select name,orderdate,cost,
      lag(orderdate,1,'1900-01-01') over(partition by name order by orderdate ) as time1, -- 上一行，有默认值
      lag(orderdate,2) over (partition by name order by orderdate) as time2 -- 上两行，无默认值，则为NULL
  from business;
  -- 按时间排序切成5块，返回第一块数据
  select * from (
      select name,orderdate,cost, ntile(5) over(order by orderdate) sorted from window_test ) t
  where sorted = 1；
  ```

  
