# etl-engine
实现从源读取数据 -> (目标数据类型转换 | 数据分发) -> 写到目标数据源 

# 产品概述
- 产品由etl-engine引擎和etl-designer云端设计器及crontab调度组成，
- etl-engine引擎负责解析etl配置文件并执行etl任务，
- etl-designer云端设计器通过拖拉拽的方式生成etl-engine引擎可识别的etl任务配置文件，
- crontab调度设计器负责按时间周期执行指定的etl任务，crontab调度还提供了查询etl任务执行日志功能，
- 三部分组成了etl解决方案，可集成到任意使用场景。

[产品详细介绍](https://pan.baidu.com/s/1TL00bs9lvi0-8zrKewFrcg?pwd=kc7a)

[高可用介绍](https://pan.baidu.com/s/1xfJ25KI4KH6ZMEW3sZ5HlA?pwd=36be)


# 资源地址
- **etl-engine下载地址**

`当前版本最后编译时间20230101`

[下载地址](https://github.com/hw2499/etl-engine/releases)


- **etl-designer设计器视频播放地址**

`etl-designer设计器支持OEM发行`（目前已经集成到[etl_crontab](https://github.com/hw2499/etl-engine/wiki/etl-crontab%E8%B0%83%E5%BA%A6)中使用）

[视频播放地址](https://www.zhihu.com/zvideo/1556673426865139712?playTime=0.0)


- **crontab调度设计器视频播放地址**

[视频播放地址](https://www.zhihu.com/zvideo/1568881056832462848?playTime=0.0)

[etl_crontab使用说明](https://github.com/hw2499/etl-engine/wiki/etl-crontab%E8%B0%83%E5%BA%A6)

#  功能特性
- 支持跨平台执行（windows,linux），只需要一个可执行文件和一个配置文件就可以运行，无需其它依赖，轻量级引擎。
- 输入输出数据源支持influxdb v1、clickhouse、prometheus、elastic、postgresql、mysql、oracle、sqlite、rocketmq、kafka、redis、excel
- 任意一个输入节点可以同任意一个输出节点进行组合，遵循pipeline模型。
- 为满足业务场景需要，支持配置文件中使用全局变量，实现动态更新配置文件功能。
- 任意一个输出节点都可以嵌入go语言脚本并进行解析，实现对输出数据流的格式转换功能。
- 支持节点级二次开发，通过配置自定义节点，并在自定义节点中配置go语言脚本，可扩展实现各种功能操作。
- 任意一个输入节点都可以通过组合数据流拷贝节点，实现从一个输入同时分支到多个输出的场景。
- 支持将各节点执行日志输出到数据库中。
- 支持跟crontab调度组合配置，实现周期性执行etl-engine任务。


# 数据流特性

- 输入输出任意组合 

![输入输出](https://i.postimg.cc/yxYHLv3y/input-output.png)
- 解析嵌入脚本语言 方便格式转换 

![支持嵌入脚本语言](https://i.postimg.cc/gJM5nn6d/script.png)
- 数据流复制 方便多路输出 

![数据流复制](https://i.postimg.cc/9FY9RMc3/copy-stream.png)

- 自定义节点 方便实现各种操作

![自定义节点](https://i.postimg.cc/Dz1gGtVZ/custom-node.png)  

- 转换节点 方便实现各种转换

![转换节点](https://i.postimg.cc/tT6xKygt/transform.png)  


# 调度集成方案


- etl_crontab与etl_engine灵活组合 

![集成方案](https://i.postimg.cc/0Q6CsFvd/crontab.png)

- etl-designer设计器 

![etl-designer设计器](https://i.postimg.cc/XYQg2fB2/etldesigner.png)

- 调度设计器 

![调度设计器](https://i.postimg.cc/j2DV5q0T/run-crontab.png)

- 调度日志 

![调度日志](https://i.postimg.cc/cHrZKBwP/joblist.png)

- Etl日志明细 

![Etl日志明细](https://i.postimg.cc/4xq1kJwt/etllogs.png)

# 使用方式
## window平台
```sh
  etl_engine.exe -fileUrl .\graph.xml -logLevel info 
 
```
## linux平台
```sh
  
  etl_engine -fileUrl .\graph.xml -logLevel info 

```
# 配置文件样例
```sh
<?xml version="1.0" encoding="UTF-8"?>
<Graph>
  <Node id="DB_INPUT_01" dbConnection="CONNECT_01" type="DB_INPUT_TABLE" desc="节点1" fetchSize="5">
      <Script name="sqlScript"><![CDATA[
		         select * from (select * from t3 limit 10)
]]></Script>
  </Node>
  <Node id="DB_OUTPUT_01" type="DB_OUTPUT_TABLE" desc="节点2" dbConnection="CONNECT_02" outputFields="f1;f2" renameOutputFields="c1;c2" outputTags="tag1;tag4"  renameOutputTags="tag_1;tag_4"  measurement="t1" rp="autogen">
  </Node>
  <!--
     <Node id="DB_OUTPUT_02" type="DB_OUTPUT_TABLE" desc="节点3" dbConnection="CONNECT_03" outputFields="f1;f2;f3"  renameOutputFields="c1;c2;c3"  batchSize="1000"  >
        <Script name="sqlScript"><![CDATA[
           insert into db1.t1 (c1,c2,c3) values (?,?,?)
    ]]></Script>
    </Node>
  -->
  <Line id="LINE_01" type="STANDARD" from="DB_INPUT_01" to="DB_OUTPUT_01" order="0" metadata="METADATA_01"></Line>
  <Metadata id="METADATA_01">
    <Field name="c1" type="string" default="-1" nullable="false"/>
    <Field name="c2" type="int" default="-1" nullable="false"/>
    <Field name="tag_1" type="string" default="-1" nullable="false"/>
    <Field name="tag_4" type="string" default="-1" nullable="false"/>
  </Metadata>
  <Connection id="CONNECT_01" dbURL="http://127.0.0.1:58080" database="db1" username="user1" password="******" token=" " org="hw"  type="INFLUXDB_V1"/>

  <Connection id="CONNECT_02" dbURL="http://127.0.0.1:58086" database="db1" username="user1" password="******" token=" " org="hw"  type="INFLUXDB_V1"/>
 <!--    <Connection id="CONNECT_04" dbURL="127.0.0.1:19000" database="db1" username="user1" password="******" type="CLICKHOUSE"/>-->
  <!--    <Connection id="CONNECT_03" dbURL="127.0.0.1:3306" database="db1" username="user1" password="******" type="MYSQL"/>-->
  <!--        <Connection id="CONNECT_03"  database="d:/sqlite_db1.db"  batchSize="10000" type="SQLITE"/>-->
</Graph>
```

# 支持节点类型
`任意一个读节点都可以输出到任意一个写节点`
##  [DB_INPUT_TABLE](./README.md#%E8%8A%82%E7%82%B9db_input_table)
`输入节点-读数据表`
##  [DB_OUTPUT_TABLE](./README.md#%E8%8A%82%E7%82%B9db_output_table)
`输出节点-写数据表`
##  [XLS_READER](./README.md#%E8%8A%82%E7%82%B9xls_reader)
`输入节点-读 excel文件`
##  [XLS_WRITER](./README.md#%E8%8A%82%E7%82%B9xls_writer)
`输出节点-写 excel文件`
## [DB_EXECUTE_TABLE](./README.md#%E8%8A%82%E7%82%B9db_execute_table)
`输入节点-执行数据库脚本`
## [OUTPUT_TRASH](./README.md#%E8%8A%82%E7%82%B9output_trash)
`输出节点-垃圾桶，没有任何输出`
## [MQ_CONSUMER](./README.md#%E8%8A%82%E7%82%B9mq_consumer)
`输入节点-MQ消费者`
## [MQ_PRODUCER](./README.md#%E8%8A%82%E7%82%B9mq_producer)
`输出节点-MQ生产者`
## [COPY_STREAM](./README.md#%E6%95%B0%E6%8D%AE%E6%B5%81%E6%8B%B7%E8%B4%9D%E8%8A%82%E7%82%B9)
`数据流拷贝节点，位于输入节点和输出节点之间，既是输出又是输入`
## [REDIS_READER](./README.md#%E8%8A%82%E7%82%B9redis_reader)
`输入节点-读 redis`
## [REDIS_WRITER](./README.md#%E8%8A%82%E7%82%B9redis_writer)
`输出节点-写 redis`
## [CUSTOM_READER_WRITER](./README.md#%E8%8A%82%E7%82%B9custom_reader_writer)
`自定义节点，通过嵌入go脚本来实现各种操作`
## [EXECUTE_SHELL](./README.md#%E8%8A%82%E7%82%B9execute_shell)
`输入节点-执行系统脚本节点`
## [CSV_READER](./README.md#%E8%8A%82%E7%82%B9csv_reader)
`输入节点-读取CSV文件节点`
## [PROMETHEUS_API_READER](./README.md#prometheus_api_reader-1)
`输入节点-读PROMETHEUS节点`
## [PROMETHEUS_EXPORTER](./README.md#prometheus_exporter-1)
`输入节点-PROMETHEUS EXPORTER节点`
## [PROMETHEUS_API_WRITER](./README.md#prometheus_api_writer-1)
`输出节点-写PROMETHEUS节点`
## [HTTP_INPUT_SERVICE](./README.md#http_input_service-1)
`输入节点-Http节点`
## [ELASTIC_READER](./README.md#elastic_reader-1)
`输入节点-读es节点`
## [ELASTIC_WRITER](./README.md#elastic_writer-1)
`输出节点-写es节点`


## 组合方式
- `任意一个输入节点都可以连接到任意一个输出节点`
- `任意一个输入节点都可以连接到一个拷贝节点`
- `一个拷贝节点可以连接到多个输出节点`
- `任意一个输入节点都可以连接到一个转换节点`
- `拷贝节点不允许连接转换节点`


# 配置说明
## 节点DB_INPUT_TABLE
`输入节点`

| 属性           | 说明               |
|--------------|------------------|
| id          | 唯一标示             |
| type         | 类型, DB_INPUT_TABLE |
| script       | sqlScript SQL语句  |
| fetchSize    | 每次读取记录数          |
| dbConnection | 数据源ID            |
| desc         | 描述               |


### 支持源类型
MYSQL、Influxdb 1x、CK、PostgreSQL、Oracle、sqlite

### 样本
```sh
  <Node id="DB_INPUT_01" dbConnection="CONNECT_01" type="DB_INPUT_TABLE" desc="节点1" fetchSize="1000">
    <Script name="sqlScript"><![CDATA[
		         select * from (select * from t4 limit 100000)
]]></Script>
  </Node>
```
## 节点XLS_READER
`输入节点`
### 读取EXCEL文件内容

| 属性           | 说明                                                            |
|--------------|---------------------------------------------------------------|
| id          | 唯一标示                                                          |
| type         | 类型, XLS_READER                                                |
| fileURL       | 文件路径+文件名称                                                     |
| startRow    | 从第几行开始读取，第1行索引是0（通常是列标题）                           |
| sheetName | 表名称                                                           |
| maxRow    | 最多读几行,*代表全部，10代表读取10行                   |
| fieldMap         | 字段映射关系，格式：field1=A;field2=B;field3=C<br/>字段名称=第几列 多个字段之间用分号分隔 |

### 样本
```shell
  <Node id="XLS_READER_01"   type="XLS_READER" desc="输入节点1"  fileURL="d:/demo/test1.xlsx" startRow="2" sheetName="人员信息" fieldMap="field1=A;field2=B;field3=C">
  </Node>
```

## 节点DB_OUTPUT_TABLE
`输出节点`

| 属性           | 说明                         | 适合                                      |
|--------------|----------------------------|-----------------------------------------|
| id          | 唯一标示                       ||
| type         | 类型, DB_OUTPUT_TABLE        ||
|script| insert、delete、update SQL语句 |ck,mysql,sqlite,postgre,oracle|
| batchSize       | 每次批提交的记录数                  | ck,mysql,sqlite,postgre,oracle <br/>注意influx以输入时的fetchSize为批提交的大小 |
| outputFields    | 输入节点读数据时传递过来的字段名称          | influx,ck,mysql,sqlite,postgre,oracle                         |
| renameOutputFields    | 输出节点到目标数据源的字段名称            | influx,ck,mysql,sqlite,postgre,oracle                         |
| dbConnection | 数据源ID                      ||
| desc         | 描述                         ||
|outputTags| 输入节点读数据时传递过来的标签名称          | influx                                  |
|renameOutputTags| 输出节点到目标数据源的标签名称            | influx                                  |
|rp| 保留策略名称                     | influx                                  |
|measurement| 表名称                        | influx                                  |
|timeOffset| 时间抖动偏移量,用于批量写入时生成不可重复的时间戳<br/>(该功能通过time.Sleep实现,建议通过嵌入脚本增加一个纳秒格式的time列,或调整你的time+tags) | influx|


## 支持目标类型
MYSQL、Influxdb 1x、CK、PostgreSQL、Oracle、sqlite


### 样本
```sh
  <Node id="DB_OUTPUT_01" type="DB_OUTPUT_TABLE" desc="写influx节点1" dbConnection="CONNECT_02" outputFields="f1;f2;f3;f4"  renameOutputFields="c1;c2;c3;c4"  outputTags="tag1;tag2;tag3;tag4"  renameOutputTags="tag_1;tag_2;tag_3;tag_4" measurement="t5" rp="autogen">
        
  </Node>
  
  <Node id="DB_OUTPUT_02" type="DB_OUTPUT_TABLE" desc="写mysql节点2" dbConnection="CONNECT_03" outputFields="time;f1;f2;f3;f4;tag1;tag2;tag3;tag4"  renameOutputFields="time;c1;c2;c3;c4;tag_1;tag_2;tag_3;tag_4" batchSize="1000" >
        <Script name="sqlScript"><![CDATA[
          insert into db1.t1 (time,c1,c2,c3,c4,tag_1,tag_2,tag_3,tag_4) values (?,?,?,?,?,?,?,?,?)
    ]]></Script>
  </Node>
```

## 节点XLS_WRITER
`输出节点`
### 写入EXCEL文件内容

| 属性           | 说明                                                |
|--------------|---------------------------------------------------|
| id          | 唯一标示                                              |
| type         |  XLS_WRITER                                    |
| fileURL       | 文件路径+文件名称                                         |
| startRow    | 从第几行开始读取  如：数字2代表是第2行 开始写数据                       |
| sheetName | 表名称                                               |
|outputFields| 输入节点传递过来的字段名称，<br/>格式：field1;field2;field3        |
| renameOutputFields         | 字段映射关系，格式：指标=B;年度=C;地区=D<br/>字段名称=第几列 多个字段之间用分号分隔 |
|metadataRow| 输出EXCEL文件中第几行输出字段名称，如：数字1代表是第1行 开始写字段名称           |
|appendRow| true代表追加记录模式，false代表覆盖模式。                         |


### 样本
```shell
  <Node id="XLS_WRITER_01"   type="XLS_WRITER" desc="输出节点2" appendRow="true"  fileURL="d:/demo/test2.xlsx" startRow="3" metadataRow="2" sheetName="人员信息" outputFields="c1;c3;tag_1"  renameOutputFields="指标=B;年度=C;地区=D"  >
    </Node>
```


## 节点DB_EXECUTE_TABLE
`输入节点`
### 执行insert ,delete ,update语句

| 属性        | 说明                          | 适合                             |
|-----------|-----------------------------|--------------------------------|
| id    | 唯一标示       ||
| type         | DB_EXECUTE_TABLE                                    |
| roolback  | 是否回滚                        | false不回滚，true回滚                |
| sqlScript | delete、update语句，多条语句之间用分号分隔 | mysql，sqlite，postgre，oracle，ck(不支持delete,update)     |
| fileURL   | 外部文件                        | fileURL优先级别高于sqlScript,两个只能用一个 |



### 样本
```sh
 <Node id="DB_EXECUTE_01" dbConnection="CONNECT_01" type="DB_EXECUTE_TABLE" desc="节点1" rollback="false" >
    <Script name="sqlScript"><![CDATA[
		         insert into t_1 (uuid,name) values (13,'aaa');
		         insert into t_1 (uuid,name) values (14,'bbb');
		         insert into t_1 (uuid,name) values (15,'ccc');
		         insert into t_1 (uuid,name) values (1,'aaa')
]]></Script>
```


## 节点OUTPUT_TRASH
`输出节点`
### 空管道，没有任何输出，适用于作为没有任何输出的节点所连接的目标节点（比如：DB_EXECUTE_TABLE节点）

### 样本
```sh
  <Node id="OUTPUT_TRASH_01"   type="OUTPUT_TRASH" desc="节点2"  >
      </Node>
```



## 节点MQ_CONSUMER
`输入节点，阻塞模式`
### mq消费者 （支持rocketmq）

| 属性         | 说明                             | 适合           |
|------------|--------------------------------|--------------|
| id         | 唯一标示                           ||
| type       | MQ_CONSUMER                    |              |
| flag       | 默认值：ROCKETMQ                   | 支持rocketmq |
| nameServer | mq服务器地址，格式：127.0.0.1:8080      |              |
| group      | mq组名称                          |              |
| topic      | 订阅主题名称                         |              |
| tag        | 标签名称，格式：*代表消费全部标签,<br/>tag_1代表只消费tag_1标签|  |


### 样本
```shell
    <Node id="MQ_CONSUMER_02" type="MQ_CONSUMER" flag="ROCKETMQ" nameServer="127.0.0.1:8080" group="group_1" topic="out_event_user_info" tag="*"></Node>
```

### mq消费者 （支持kafka）

| 属性         | 说明                        | 适合      |
|------------|---------------------------|---------|
| id         | 唯一标示                      ||
| type       | MQ_CONSUMER               |         |
| flag       | 默认值：KAFKA                 | 支持kafka |
| nameServer | mq服务器地址，格式：127.0.0.1:8080 |         |
| group      | mq组名称                     |         |
| topic      | 订阅主题名称                    |         |
| listenerFlag        | 1是按分区进行监听 ; 2是按单通道进行监听,topic可以是多个|         |

### 样本
```shell
 <Node id="MQ_CONSUMER_03" type="MQ_CONSUMER" flag="KAFKA" nameServer="127.0.0.1:18081" group="group_10" topic="out_event_user_info" listenerFlag="2"></Node>

```




## 节点MQ_PRODUCER
`输出节点`
### mq生产者 （支持rocketmq）

| 属性         | 说明                        | 适合           |
|------------|---------------------------|--------------|
| id         | 唯一标示                      ||
| type       | MQ_PRODUCER               |              |
| flag       | 默认值：ROCKETMQ              | 支持rocketmq |
| nameServer | mq服务器地址，格式：127.0.0.1:8080 |              |
| group      | mq组名称                     |              |
| topic      | 订阅主题名称                    |              |
| tag        | 标签名称，格式：tag_1             |  |
| sendFlag        | 发送模式,1是同步；2是异步；3是单向       |  |
|outputFields| 输入节点传递过来的字段名称，<br/>格式：field1;field2;field3 多个字段之间用分号分隔        |
| renameOutputFields         | 字段映射关系，格式：field1;field2;field3  多个字段之间用分号分隔 |

### 样本
```shell
    <Node id="MQ_PRODUCER_01" type="MQ_PRODUCER" flag="ROCKETMQ" nameServer="127.0.0.1:8080" group="group_11" topic="out_event_system_user" tag="tag_1"
          sendFlag="3" outputFields="time;tag_1;c2"  renameOutputFields="时间;设备;指标" >
    </Node>
```

### mq生产者 （支持kafka）

| 属性         | 说明                                                     | 适合      |
|------------|--------------------------------------------------------|---------|
| id         | 唯一标示                                                   ||
| type       | MQ_PRODUCER                                            |         |
| flag       | 默认值：KAFKA                                              | 支持kafka |
| nameServer | mq服务器地址，格式：127.0.0.1:8080                              |         |
| topic      | 订阅主题名称                                                 |         |
| isPartition        | true代表指定分区发消息;false代表随机分区发消息                           |         |
| sendFlag        | 发送模式,1是同步；2是异步                                   |         |
|outputFields| 输入节点传递过来的字段名称，<br/>格式：field1;field2;field3 多个字段之间用分号分隔 |
| renameOutputFields         | 字段映射关系，格式：field1;field2;field3  多个字段之间用分号分隔            |

### 样本
```shell

     <Node id="MQ_PRODUCER_02" type="MQ_PRODUCER" flag="KAFKA" nameServer="127.0.0.1:18081"  topic="out_event_system_user"
          sendFlag="1" outputFields="Offset;Partition;Topic;Value"  renameOutputFields="Offset;Partition;Topic;Value" >
    </Node>
```

## 数据流拷贝节点
`将一个输入节点的数据流输出到多个分支输出节点`

| 属性         | 说明          | 适合      |
|---|-------------|---|
| id         | 唯一标示        ||
| type       | COPY_STREAM |         |


### 样本
```shell
  <Node id="COPY_STREAM_01" type="COPY_STREAM" desc="数据流拷贝节点" ></Node>
  <Line id="LINE_01" type="STANDARD" from="DB_INPUT_01" to="COPY_STREAM_01" order="1" metadata="METADATA_01" ></Line>
  <Line id="LINE_02" type="COPY" from="COPY_STREAM_01:0" to="DB_OUTPUT_01" order="2" metadata="METADATA_01"></Line>
  <Line id="LINE_03" type="COPY" from="COPY_STREAM_01:1" to="DB_OUTPUT_02" order="2" metadata="METADATA_02"></Line>
```


## 节点REDIS_READER
`输入节点`

| 属性         | 说明             | 适合    |
|---|---|-------|
| id         | 唯一标示           ||
| type       | REDIS_READER   |       |
| nameServer       | 127.0.0.1:6379 |       |
| password       | ******         |       |
| db       | 0              | 数据库ID |
| isGetTTL       | true或false 是否读取ttl信息    |       |
| keys       | 读取的KEY，多个KEY之间用分号分隔    |  目前只支持读取string,int,float类型内容     |


### 样本
```shell

  <Node id="REDIS_READER_01"   type="REDIS_READER" desc="输入节点1" 
  nameServer="127.0.0.1:6379" password="******" db="0" isGetTTL="true" keys="a1;a_1" ></Node>
```

## 节点REDIS_WRITER
`输出节点，因key名称不可重复，所以只适合将读节点中的最后一行记录进行写入操作`


| 属性         | 说明                   | 适合                         |
|---|----------------------|----------------------------|
| id         | 唯一标示                 ||
| type       | REDIS_WRITER         |                            |
| nameServer       | 127.0.0.1:6379       |                            |
| password       | ******               |                            |
| db       | 0                    | 数据库ID                      |
| isGetTTL       | true或false 是否写入ttl信息 |                            |
| outputFields       |   | 目前只支持写string,int,float类型内容 |
| renameOutputFields       |   | 目前只支持写string,int,float类型内容 |

### 样本
```shell

  <Node id="REDIS_WRITER_01"   type="REDIS_WRITER" desc="输出节点1"  nameServer="127.0.0.1:6379" password="******" db="1" 
  isGetTTL="true" outputFields="a1;a_1"  renameOutputFields="f1;f2"  ></Node>
```

## 节点CUSTOM_READER_WRITER
`自定义节点，可以通过嵌入GO脚本实现各种操作`


| 属性         | 说明                   | 适合                         |
|---|----------------------|----------------------------|
| id         | 唯一标示                 ||
| type       | CUSTOM_READER_WRITER         |                            |



## 节点EXECUTE_SHELL
`输入节点-执行系统脚本节点`


| 属性         | 说明              | 适合                                           |
|---|-----------------|----------------------------------------------|
| id         | 唯一标示            ||
| type       | EXECUTE_SHELL   |                                              |
| fileURL       | 外部脚本文件位置        | fileURL与Script两者只能出现一个，同时出现时fileURL优先于Script |
| Script       | 脚本内容            |                                              |
| outLogFileURL       | 控制台输出内容到指定的日志文件 |                                              |


### 样本
```shell
<Node id="EXECUTE_SHELL_01"  type="EXECUTE_SHELL" desc="节点1"  _fileURL="d:/test1.bat" outLogFileURL="d:/test1_log.txt">
    <Script><![CDATA[
    c:
    dir/w
]]></Script>
  </Node>
```

## 节点CSV_READER
`输入节点-读取CSV文件节点`


| 属性         | 说明                         | 适合                       |
|---|----------------------------|--------------------------|
| id         | 唯一标示                       ||
| type       | CSV_READER                 |                          |
| fileURL       | CSV文件位置                    |                          |
| fetchSize       | 每次读取到内存中的批量数               | 如：可配合influxdb中每次批量提交的记录数，曾测试1W多条数据123个字段，配置100，入库时间为15秒 |
| startRow       | 从第几行开始读数据,默认0代表第1行         | 一般0是第一行列名称               |
| fields       | 定义输出的字段名称，多个字段间用分号分隔       | field1;field2;field3     |
| fieldsIndex       | 定义输出的列，默认0代表第1列，多个字段间用分号分隔；配置成-1代表按顺序读取所有字段 | "2;3;4"                  |

### 样本

```shell
  <Node id="CSV_READER_01"   type="CSV_READER" desc="输入节点1" fetchSize="5"  fileURL="d:/demo2.csv" startRow="1" fields="field1;field2;field3"  fieldsIndex="0;3;4">
  </Node>

```

## PROMETHEUS_API_READER
`输入节点-读PROMETHEUS节点`

| 属性     | 说明                    | 适合                      |
|--------|-----------------------|-------------------------|
| id     | 唯一标示                  ||
| type   | PROMETHEUS_API_READER |                         |
| url    | prometheus服务器地址       | 如：http://127.0.0.1:9090 |
| Script | 查询API内容，只支持/api/v1/query <br/>和 /api/v1/query_range           | 如：/api/v1/query?query=my_device_info{deviceCode="设备编码000"}[1d]|

**注意：查询返回的结果集中，__name__是度量名称；__TIME__是prometheus入库时的时间戳；__VALUE__是prometheus的value**

```shell
    <Node id="PROMETHEUS_API_READER_1" type="PROMETHEUS_API_READER"  url="http://127.0.0.1:9090" >
        <Script name="sqlScript">
            <![CDATA[
            /api/v1/query?query=my_device_info{deviceCode="设备编码000"}[1d]
            ]]>
		</Script>
    </Node>
     <Node id="DB_OUTPUT_TABLE_1" type="DB_OUTPUT_TABLE" batchSize="10" dbConnection="CONNECT_1" desc="" outputFields="__name__;address;deviceCode;__TIME__;__VALUE__" renameOutputFields="f1;f2;f3;f4;f5"  >
        <Script name="sqlScript">
            <![CDATA[insert into 
t_prome_info_bk
(f1,f2,f3,f4,f5)
values (?,?,?,?,?)]]>
        </Script>
```


## PROMETHEUS_EXPORTER
`输入节点-PROMETHEUS EXPORTER节点`

| 属性     | 说明                  | 适合                    |
|--------|---------------------|-----------------------|
| id     | 唯一标示                ||
| type   | PROMETHEUS_EXPORTER |                       |
| exporterAddr    | exporter地址, IP:PORT | 如：  :10000            |
| exporterMetricsPath    | exporter路径,         | 如：/EtlEngineExport    |
| metricName    | 度量名称                | 如：Etl_Engine_Exporter |
| metricHelp    | 度量描述                | 如：样本                  |
| labels    | 标签名称                | 如：deviceCode;address;desc                  |

```shell
    <Node id="PROMETHEUS_EXPORTER_1" type="PROMETHEUS_EXPORTER"  
	    exporterAddr=":10000" exporterMetricsPath="/EtlEngineExport" 
	    metricName="Etl_Engine_Exporter" metricHelp="Etl_Engine_Exporter样本" 
	    labels="deviceCode;address;desc">
	</Node>
```

**prometheus配置文件增加如下内容：**
```shell
  - job_name: "etlengine_exporter"
    metrics_path: "/EtlEngineExport" 
    static_configs:
      - targets: ["127.0.0.1:10000"]
```


**同时会暴露一个服务地址/pushDataService用于生成数据，postman调试细节如下：**
```shell
	 
	 POST 方式
	 URL  http://127.0.0.1:10000/pushDataService , 
	 Body x-www-form-urlencoded
	 参数:
		"jsondata":{
			"labels":{"deviceCode":"设备编码001","address":"南关区","desc":"最大值"},
			"value":100
		}
```

**输出的数据流中会自动添加两个字段，__name__是度量名称，__VALUE__是prometheus的value，**



## PROMETHEUS_API_WRITER
`输出节点-写PROMETHEUS节点`


| 属性     | 说明                                        | 适合                      |
|--------|-------------------------------------------|-------------------------|
| id     | 唯一标示                                      ||
| type   | PROMETHEUS_API_WRITER                     |                         |
| url    | prometheus服务器地址                           | 如：http://127.0.0.1:9090 |
| metric | 度量名称                                      | |
| outputFields | 输入节点传递过来的字段名称                             | |
| renameOutputFields | prometheus入库时对应的标签名称，数据项与outputFields各项对应 | |
| valueField | prometheus入库时对应的value，数据项与输入节点中已存在的字段名称对应 | |


```shell
  <Node id="DB_INPUT_TABLE_1" type="DB_INPUT_TABLE" fetchSize="1000" dbConnection="CONNECT_1"  >
        <Script name="sqlScript">
            <![CDATA[select f2,f3,f4 from t_prome_info ]]>
        </Script>
    </Node>
    <Node id="PROMETHEUS_API_WRITER_1" type="PROMETHEUS_API_WRITER" url="http://127.0.0.1:9090" metric="my_device_info" outputFields="f2;f3" renameOutputFields="deviceCode;address" valueField="f4" >
	</Node>
```

## HTTP_INPUT_SERVICE
`输入节点-Http节点,阻塞模式`

| 属性     | 说明                 | 适合  |
|--------|--------------------|-----|
| id     | 唯一标示               ||
| type   | HTTP_INPUT_SERVICE |     |
| serviceIp   | 绑定HTTP/HTTPS服务的IP  |     |
| servicePort   | 绑定HTTP/HTTPS服务的端口      |     |
| serviceName   | 对外暴露的服务名称          | 默认：etlEngineService |
| serviceCertFile   | HTTPS服务证书文件位置      |     |
| serviceKeyFile   | HTTPS服务秘钥文件位置      |     |


```shell
	<Node id=""
		type="HTTP_INPUT_SERVICE"
		serviceIp=""
		servicePort="8081"
		serviceName="etlEngineService"
		serviceCertFile=""
		serviceKeyFile="" >
	</Node>
	
	
	postman调试:
         http://127.0.0.1:8081/etlEngineService
	 POST 方式,URL: /etlEngineService , Body:x-www-form-urlencoded
	 参数：
		"jsondata":{
			"rows":[
				{"deviceCode":"设备编码001","address":"朝阳区","desc":"最大值","value":20},
				{"deviceCode":"设备编码002","address":"朝阳区","desc":"最大值","value":18}
			]
		}
     注意:必须传递KEY为rows的数组结构
```

## ELASTIC_READER
`输入节点-读es节点`

| 属性           | 说明             | 适合  |
|--------------|----------------|-----|
| id           | 唯一标示           ||
| type         | ELASTIC_READER |     |
| index        | 索引名称           |     |
| sourceFields | 结果集中输出的字段名称    |     |
| fetchSize    | 每次读取记录数        |     |
| Script标签     | es查询语法         |     |

### 样本

```shell
   <Node id="ELASTIC_READER_01" dbConnection="CONNECT_02"   
	type="ELASTIC_READER" desc="节点2"  sourceFields="custom_type;username;desc;address" fetchSize="2" >
        <Script name="sqlScript"><![CDATA[
            {
                  "query" : {
                       "bool":{
                          "must":[
                              //{
                                // "term": { "username.keyword": "王先生"  }
                             //     "match": { "username": ""  }
                             //  },
                               {
                                 "term":   {   "custom_type":"t_user_info"  }
                               }
                          ]
            
                       }
                    }
                }
        ]]></Script>
   </Node>
   
    <Connection id="CONNECT_02" type="ELASTIC" dbURL="http://127.0.0.1:9200" database="es_db3" username="elastic" password="******" />
  
```

## ELASTIC_WRITER
`输出节点-写es节点`


| 属性                 | 说明                                                                                                                                                                   | 适合  |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----|
| id                 | 唯一标示                                                                                                                                                                 ||
| type               | ELASTIC_WRITER                                                                                                                                                       |     |
| index              | 索引名称                                                                                                                                                                 |     |
| idType             | 主键输出方式： 1 代表不指定id,由es系统自己生成20位GUID;<br/>2 代表由idExpress中指定输出的字段名称，从上一个节点的renameOutputFields中进行匹配相同的字段名称的值;<br/>3 代表由idExpress中指定表达式配置，_HW_UUID32 表达式 代表按32位UUID自动生成主键 |     |
| idExpress          | 如：idType配置3；该参数配置 _HW_UUID32                                                                                                                                         |     |
| outputFields       | 输入节点读数据时传递过来的字段名称                                                                                                                                                    |    按顺序输出字段内容，不按字段名称 |
| renameOutputFields | 输出节点到目标数据源的字段名称                                                                                                                                                      |   按顺序输出字段内容，不按字段名称 |

### 样本

```shell
  <Node id="DB_INPUT_01" dbConnection="CONNECT_01" type="DB_INPUT_TABLE" desc="节点1" fetchSize="2">
    <Script name="sqlScript"><![CDATA[
		        SELECT "t_user_info" AS custom_type,uname, udesc,uaddress,uid FROM t_u_info
]]></Script>
  </Node>

    <Node id="ELASTIC_WRITER_01" dbConnection="CONNECT_02"  type="ELASTIC_WRITER" desc="节点2"  
        outputFields="custom_type;uname;udesc;uaddress;uid" 
        renameOutputFields="custom_type;username;desc;address;uid"
        idType="3" 
        idExpress="_HW_UUID32">
      </Node>
      
       <Line id="LINE_01" type="STANDARD" from="DB_INPUT_01" to="ELASTIC_WRITER_01" order="0" metadata="METADATA_01"></Line>

  <Metadata id="METADATA_01">
    <Field name="custom_type" type="string" default="-1" nullable="false"/>
    <Field name="username" type="string" default="-1" nullable="false"/>
    <Field name="desc" type="string" default="-1" nullable="false"/>
    <Field name="address" type="string" default="-1" nullable="false"/>
    <Field name="uid" type="string" default="-1" nullable="false"/>
  </Metadata>
  
  <Connection id="CONNECT_02" type="ELASTIC" dbURL="http://127.0.0.1:9200" database="es_db3" username="elastic" password="******" />
  
      
```







## 元数据Metadata

元数据文件定义目标数据格式（如输出节点中定义的renameOutputFields或renameOutputTags所对应的字段名称及字段类型）
`outputFields是输入节点中数据结果集中的字段名称，
将outputFields定义的字段转换成renameOutputFields定义的字段，其renameOutputFields转换格式通过元数据文件来定义。`

| 属性    | 说明         | 适合                                               |
|-------|------------|--------------------------------------------------|
| id    | 唯一标示       ||
| field |            |                                                  |
| name  | 输出数据源的字段名称 | renameOutputFields,<br/>renameOutputTags         |
| type  | 输出数据源的字段类型 | string,int,int32,float,<br/>str_timestamp,decimal,<br/>datetime,timestamp  |
| default | 默认值        | 当nullable为false时，如果输出值为空字符串，则可以通过default来指定输出的默认值 |
| nullable| 是否允许为空     | false不允许为空，必须和default配合使用。true允许为空。             |


## 数据源Connection

| 属性   | 说明       | 适合                 |
|---|----------|--------------------|
| id   | 唯一标示     |  |
| type   | 数据源类型    |INFLUXDB_V1、MYSQL、CLICKHOUSE、SQLITE、POSTGRES、ORACLE、ELASTIC|
| dbURL | 连接地址     | ck,mysql,influx,postgre,oracle,elastic    |
| database   | 数据库名称    | ck,mysql,influx,sqlite,postgre,oracle,elastic    |
| username   | 用户名称     | ck,mysql,influx,postgre,oracle,elastic    |
| password   | 密码       | ck,mysql,influx,postgre,oracle,elastic    |
| token   | token名称  | influx 2x          |
| org   | 机构名称     | influx 2x          |
| rp   | 数据保留策略名称 | influx 1x          |


## Graph

| 属性       | 说明          | 适合                                  |
|----------|-------------|-------------------------------------|
| runMode       | 1串行模式;2并行模式 | 默认推荐使用并行模式,<br/>如果需要各流程排序执行,可使用串行模式 |



## 连接线Line

| 属性       | 说明                                                         | 适合                                      |
|----------|------------------------------------------------------------|-----------------------------------------|
| id       | 唯一标示                                                       ||
| from     | 输入节点唯一标示                                                   ||
| to       | 输出节点唯一标示                                                   ||
| type     | STANDARD 标准,一进一出,COPY 复制数据流,中间环节复制数据                ||
| order    | 串行排序号,按正整数升序排列,在graph属性runMode为1时,<br/>通过配置0,1,2这种方式实现串行执行 ||
| metadata | 目标元数据ID                                                    ||



# 支持配置全局变量
### 通过命令行方式传递全局变量

```shell
etl_engine -fileUrl ./global6.xml -logLevel debug arg1="d:/test3.xlsx" arg2=上海
```
其中 `arg1`和`arg2`是从命令行传递进来的全局变量

- ### 配置文件中引用全局变量

```shell
    <Node id="DB_INPUT_01" dbConnection="CONNECT_01" type="DB_INPUT_TABLE" desc="节点1" fetchSize="500">
     <Script name="sqlScript"><![CDATA[
		         select * from (select * from t5 where tag_1='${arg2}' limit 1000)
    ]]></Script>

  <Node id="XLS_WRITER_01"   type="XLS_WRITER" desc="输出节点2" appendRow="true"  fileURL="${arg1}"  startRow="3" metadataRow="2" sheetName="人员信息" outputFields="c1;c3;tag_1"  renameOutputFields="指标=B;年度=C;地区=D"  >
 
```
配置文件中`${arg1}` 会在服务运行时通过命令行参数arg1的值`d:/test3.xlsx`被替换掉<br>
配置文件中`${arg2}` 会在服务运行时通过命令行参数arg2的值 `上海` 被替换掉

- ### 内置变量说明

为方便生成固定格式化内容，系统内置了常用变量，方便用于配置全局变量时动态替换变量值。
内置变量前缀 `_HW_`
1. 时间变量
   <br>格式： `_HW_YYYY-MM-DD hh:mm:ss.SSS`
   <br>输出当前系统时间，如： 2022-01-02 19:33:06.108
   <br>**注意空格通过0x32进行转义，因此正确传递方式是**
   <br>`_HW_YYYY-MM-DD0x32hh:mm:ss.SSS`
   ```shell
   YYYY 输出四位年 2022
   MM 输出两位月 01
   DD 输出两位日 02
   hh 输出两位小时 19
   mm 输出两位分钟 33
   ss 输出两位秒 06
   .SSS 输出一个前缀.和三位毫秒 .108
   ```
以上部分可随意组合，如：`_HW_YYYYMM` ，输出202201

2. 时间位移变量
 <br>**在原有时间变量基础上，大写Z字符代表对时、分钟、秒的加减位移。**

   <br>如格式： `_HW_YYYY-MM-DD hh:mm:ss.SSSZ2h45m`
   <br>输出当前时间累加2小时45分钟后的时间。
   <br>如格式：`_HW_YYYY-MM-DD hh:mm:ssZ-24h10m`
   <br>大写字符Z后面跟随负数可实现减位移。
   <br>输出当前时间减少24小时10分钟后的时间。
   <br>支持的时间频率单位如下：
   <br>"ns", "us" (or "µs"), "ms", "s", "m", "h"

   <br>**在原有时间变量基础上，小写z字符代表对年、月、日的加减位移。**

   <br>如格式：`_HW_YYYY-MM-DD hh:mm:ssz1,2,3`
   <br>输出当前时间累加1年2个月3天
   <br>如格式：`_HW_YYYY-MM-DD hhz-1,-2,-3`
   <br>输出当前时间减少1年2个月3天


3. 时间戳变量
   <br>格式： `_HW_timestamp10`
   <br>输出当前系统10位时间戳，如：1669450716
   <br>格式： `_HW_timestamp13`
   <br>输出当前系统13位时间戳，如：1669450716142
   <br>格式： `_HW_timestamp16`
   <br>输出当前系统16位时间戳，如：1669450716142516
   <br>格式： `_HW_timestamp19`
   <br>输出当前系统19位时间戳，如：1669450716142516700



4. UUID变量
   <br>格式： `_HW_uuid32`
   <br>输出32位UUID，如：D54C3C7163844E4DB4F073E8EEC83328

- ### 通过命令行方式传递内置变量

```shell
etl_engine -fileUrl ./global6.xml -logLevel debug arg1=_HW_YYYY-MM-DD0x32hh:mm:ss.SSS arg2=_HW_YYYY-MM-DD
```

- ### 配置文件中引用内置变量

```shell
    <Node id="DB_INPUT_01" dbConnection="CONNECT_01" type="DB_INPUT_TABLE" desc="节点1" fetchSize="500">
     <Script name="sqlScript"><![CDATA[
		         select * from (select * from t5 where tag_1='${arg1}' limit 1000)
    ]]></Script>

  <Node id="XLS_WRITER_01"   type="XLS_WRITER" desc="输出节点2" appendRow="true"  fileURL="${arg2}.xlsx" _fileURL="d:/demo/test2.xlsx" startRow="3" metadataRow="2" sheetName="人员信息" outputFields="c1;c3;tag_1"  renameOutputFields="指标=B;年度=C;地区=D"  >
 
```

# 支持解析嵌入go语言
可以在任意一个输出节点的 `<BeforeOut></BeforeOut>` 标签内嵌入自己的业务逻辑，[更多介绍](https://github.com/hw2499/etl-engine/wiki/%E5%B5%8C%E5%85%A5%E8%84%9A%E6%9C%AC%E5%BC%80%E5%8F%91)

- ### 增加字段
`可以增加多个字段，并赋予默认值`
```shell
package ext
import (
	"errors"
	"fmt"
	"strconv"
)
func RunScript(dataValue string) (result string, topErr error) {
	newRows := ""
	rows := gjson.Get(dataValue, "rows")
	for index, row := range rows.Array() {
	  	//tmpStr, _ := sjson.Set(row.String(), "addCol1", time.Now().Format("2006-01-02 15:04:05.000"))
		tmpStr, _ := sjson.Set(row.String(), "addCol1", "1")
		tmpStr, _ = sjson.Set(tmpStr, "addCol2", "${arg2}")
		newRows, _ = sjson.SetRaw(newRows, "rows.-1", tmpStr)
	}
	return newRows, nil
}

```
- ### 合并字段
`可以将多个字段合并为一个字段`
```shell
package ext
import (
	"errors"
	"fmt"
	"strconv"
)
func RunScript(dataValue string) (result string, topErr error) {
	newRows := ""
	rows := gjson.Get(dataValue, "rows")
	for index, row := range rows.Array() {
		area := gjson.Get(row.String(),"tag_1").String()
		year := gjson.Get(row.String(),"c3").String()
		tmpStr, _ := sjson.Set(row.String(), "tag_1", area + "_" + year)
		newRows, _ = sjson.SetRaw(newRows, "rows.-1", tmpStr)
	}
	return newRows, nil
}

```
- ### 完整样本
```
<?xml version="1.0" encoding="UTF-8"?>
<Graph>
 
  <Node id="CSV_READER_01"   type="CSV_READER" desc="输入节点1" fetchSize="500"  fileURL="d:/demo.csv" startRow="1" fields="field1;field2;field3;field4"  fieldsIndex="0;1;2;3"  >
  </Node>
 
    <Node id="OUTPUT_TRASH_01"   type="OUTPUT_TRASH" desc="节点2"  >
        <BeforeOut>
            <![CDATA[
package ext
import (
	"errors"
	"fmt"
	"strconv"
	"strings"
	"time"
	"github.com/tidwall/gjson"
	"github.com/tidwall/sjson"
	"etl-engine/etl/tool/extlibs/common"
	"io/ioutil"
	"os"
)
func RunScript(dataValue string) (result string, topErr error) {
	defer func() {
		if topLevelErr := recover(); topLevelErr != nil {
			topErr = errors.New("RunScript 捕获致命错误" + topLevelErr.(error).Error())
		} else {
			topErr = nil
		}
	}()
	newRows := ""
	GenLine(dataValue,"db1","autogen","t13","field2","field3;field4")
	return newRows, nil
}

//接收的是JSON
func GenLine(dataValue string, db string, rp string, measurement string, fields string, tags string) error {
	head := "# DML\n# CONTEXT-DATABASE: " + db + "\n# CONTEXT-RETENTION-POLICY: " + rp + "\n\n"
	line := ""
    fieldLine := ""
    tagLine := ""
	_t_ := strings.Split(tags, ";")
	_f_ := strings.Split(fields, ";")
	rows := gjson.Get(dataValue, "rows")
	for _, row := range rows.Array() {
        fieldLine = ""
        tagLine = ""
		for i1 := 0; i1 < len(_t_); i1++ {
			tagValue := gjson.Get(row.String(), _t_[i1])
			tagLine = tagLine + _t_[i1] + "=\"" + tagValue.String() + "\","
		}
		tagLine = tagLine[0 : len(tagLine)-1]
		for i1 := 0; i1 < len(_f_); i1++ {
			fieldValue := gjson.Get(row.String(), _f_[i1])
			fieldLine = fieldLine + _f_[i1] + "=" + fieldValue.String() + ","
		}
		fieldLine = fieldLine[0 : len(fieldLine)-1]

		if len(tagLine) > 0 && len(fieldLine) > 0 {
		    line = line + measurement + "," + tagLine + " " + fieldLine + " " + strconv.FormatInt(time.Now().Add(500*time.Millisecond).UnixNano(), 10) + "\n"
        } else {
            
            if len(fieldLine) > 0 {
                line = line + measurement + "," + fieldLine + " " + strconv.FormatInt(time.Now().Add(500*time.Millisecond).UnixNano(), 10) + "\n"
            }
        }

	}

	if len(line) > 0 {
		txt := head + line
		fileName := "d:/"+strconv.FormatInt(time.Now().UnixNano(), 10)
		WriteFileToDB(fileName, txt)
		err1 := os.Remove(fileName)
			if err1 != nil {
				fmt.Println("删除临时文件失败：", fileName)
				return err1
			}
	}
	return nil
}
func WriteFileToDB(fileName string, txt string) {

	buf := []byte(txt)
	err := ioutil.WriteFile(fileName, buf, 0666)
	if err != nil {
		fmt.Println("写入文件失败：", err)
		return
	} else {
		cmdLine := "D:/software/influxdb-1.8.10-1/influx.exe  -import -path=" + fileName + " -host 127.0.0.1 -port 58086 -username u1 -password 123456 -precision=ns"
		//fmt.Println("cmdLine:",cmdLine)
		common.Command3("GB18030", "cmd", "/c", cmdLine)

	}
}

              ]]>
        </BeforeOut>

    </Node>
    
  <Line id="LINE_01" type="STANDARD" from="CSV_READER_01" to="OUTPUT_TRASH_01" order="0" metadata="METADATA_03">线标注</Line>
    <Metadata id="METADATA_03">
        <Field name="field1" type="string" default="-1" nullable="false"/>
        <Field name="field2" type="string" default="-1" nullable="false"/>
        <Field name="field3" type="string" default="-1" nullable="false"/>
        <Field name="field4" type="string" default="-1" nullable="false"/>
    </Metadata>
   
</Graph>


```





# 合作模式
<details><summary>欢迎对接合作</summary>
 

#### etl-engine 全行业可接...

    ```
     @auth Mr Huang
     vx:weigeonlyyou
    ```

</details> 


