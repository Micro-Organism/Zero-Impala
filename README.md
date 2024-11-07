# Zero-Impala
Zero-Impala
# 1. 概述
## 1.1. 简介
> Impala是Cloudera公司主导开发的新型查询系统，它提供SQL语义，能查询存储在Hadoop的HDFS和HBase中的PB级大数据。已有的Hive系统虽然也提供了SQL语义， 
> 但由于Hive底层执行使用的是MapReduce引擎，仍然是一个批处理过程，难以满足查询的交互性。相比之下，Impala的最大特点也是最大卖点就是它的快速。
## 1.2. 组成
> 1. 客户端：包括JDBC、ODBC、Hue、Impala Shell等，用于执行查询或完成管理任务； 
> 2. Hive Metastore：存储可用于Impala数据的信息，包括可用数据库及其结构。当执行Impala Sql语句进行schema对象的创建、修改及删除，或加载数据到表中等操作时，相关元数据的变化，通过单独的catalog服务自动广播到所有Impala节点； 
> 3. Cloudera Impala（Impalad进程）：运行于数据节点的Impala程序，用于协调和执行查询。每一个Impala的实例可以获取、解析以及协调Impala客户端传来的查询。查询是被分布到各Impala节点间，这些节点作为workers，并行执行查询片段； 
> 4. HDFS、HBase、kudu：数据的实际存储位置。
## 1.3. impala查询执处理过程
> 1. 用户程序通过JDBC、ODBC、Impala Shell等Impala 客户端发送Sql语句给Impala； 
> 2. 用户程序连接到集群中任意Impalad进程，这一进程作为整个查询的协调器； 
> 3. Impala解析、分析查询，确定哪些任务由集群中哪一Impalad实例执行，并生成最优执行计划； 
> 4. Impalad实例访问对应HDFS、HBase服务，获取数据； 
> 5. 每一个Impalad实例将数据返回给协调器Impalad，由其发送结果给客户端。

# 2. 环境
## 2.1. install kudu

详见代码仓库kudu模块里面的docker目录

## 2.2. install impala
```shell
docker run -d --name kudu-impala --add-host  kudu-master-1:{ip} --add-host  kudu-master-2:{ip} --add-host  kudu-master-3:{ip} -p 21000:21000 -p 21050:21050 -p 25000:25000 -p 25010:25010 -p 25020:25020   --memory=4096m apache/kudu:impala-latest impala
```
需要注意增加主机映射关系，不然impala找不带kudu的机器。
访问http://172.24.4.35:25000/

## 2.3. run impala-shell
```shell
docker exec -it kudu-impala impala-shell
```
### 2.3.1. Create a Kudu Table
Now that you are in an impala-shell that is connected to Impala you can use an Impala DDL statement to create a Kudu table.
```
CREATE TABLE my_first_table
(
id BIGINT,
name STRING,
PRIMARY KEY(id)
)
PARTITION BY HASH PARTITIONS 4
STORED AS KUDU;

DESCRIBE my_first_table;
```
### 2.3.2. Insert and Modify Data
With my_first_table created you can now use Impala DML statements to INSERT, UPDATE, UPSERT, and DELETE data.
```
-- Insert a row.
INSERT INTO my_first_table VALUES (99, "sarah");
SELECT * FROM my_first_table;

-- Insert multiple rows.
INSERT INTO my_first_table VALUES (1, "john"), (2, "jane"), (3, "jim");
SELECT * FROM my_first_table;

-- Update a row.
UPDATE my_first_table SET name="bob" where id = 3;
SELECT * FROM my_first_table;

-- Use upsert to insert a new row and update another.
UPSERT INTO my_first_table VALUES (3, "bobby"), (4, "grant");
SELECT * FROM my_first_table;

-- Delete a row.
DELETE FROM my_first_table WHERE id = 99;
SELECT * FROM my_first_table;

-- Delete multiple rows.
DELETE FROM my_first_table WHERE id < 3;
SELECT * FROM my_first_table;
```

## 2.4. install hue
```shell
docker run -it -p 8888:8888 gethue/hue:latest
```
### 2.4.1. 拷贝配置文件出来
```shell
docker cpb72dfc588c76:/usr/share/hue/desktop/conf/z-hue-overrides.ini ./z-hue-overrides.ini
```
编辑文件内容
```
[[database]]
engine=mysql
host=xxx.xxx.xxx.xxx
port=3306
user=xxx
password=xxx
name=database

[impala]
server_host=xxx.xxx.xxx.xxx
server_port=21050
```
### 2.4.2. 将修改好的文件放回去
```shell
docker cp./z-hue-overrides.ini b72dfc588c76:/usr/share/hue/desktop/conf/z-hue-overrides.ini
```
重启
```shell
docker stop ${container_id}
docker start ${container_id}
```
访问http://172.24.4.35:8888/

# 3. 功能

# 4. 其他

# 5. 参考