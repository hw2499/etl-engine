# etl-engine
实现从源读取数据 -> 目标数据类型转换 -> 写到目标数据源 

# 产品概述
- 产品由etl-engine引擎和etl-designer云端设计器及crontab调度组成，
- etl-engine引擎负责解析etl配置文件并执行etl任务，
- etl-designer云端设计器通过拖拉拽的方式生成etl-engine引擎可识别的etl任务配置文件，
- crontab调度设计器负责按时间周期执行指定的etl任务，crontab调度还提供了查询etl任务执行日志功能，
- 三部分组成了etl解决方案，可集成到任意使用场景。

# 资源地址
- **etl-engine下载地址**

`当前版本最后编译时间20220827`

[下载地址](https://github.com/hw2499/etl-engine/releases/tag/v1.0.0)



- **etl-designer设计器在线使用及视频播放地址**

`etl-designer设计器支持OEM发行`

[视频播放地址](https://www.zhihu.com/zvideo/1556673426865139712?playTime=0.0)

[在线使用地址](https://hw2499.github.io/etldesigner/)

- **调度设计器视频播放地址**

[视频播放地址](https://www.zhihu.com/zvideo/1568881056832462848?playTime=0.0)

#  功能特性
- 支持跨平台执行（windows,linux），只需要一个可执行文件和一个配置文件就可以运行，无需其它依赖，轻量级引擎。
- 输入输出数据源支持influxdb v1、clickhouse、mysql、sqlite、rocketmq、kafka、redis、excel
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


- run_crontab与run_graph灵活组合 

![集成方案](https://i.postimg.cc/qqHT8sN4/crontab.png)

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
  run_graph.exe -fileUrl .\graph.xml -logLevel info 
 
```
## linux平台
```sh
  
  run_graph -fileUrl .\graph.xml -logLevel info 

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
##  DB_INPUT_TABLE
`输入节点-读数据表`
##  DB_OUTPUT_TABLE
`输出节点-写数据表`
##  XLS_READER
`输入节点-读 excel文件`
##  XLS_WRITER
`输出节点-写 excel文件`
## DB_EXECUTE_TABLE
`输入节点-执行数据库脚本`
## OUTPUT_TRASH
`输出节点-垃圾桶，没有任何输出`
## MQ_CONSUMER
`输入节点-MQ消费者`
## MQ_PRODUCER
`输出节点-MQ生产者`
## COPY_STREAM
`数据流拷贝节点，位于输入节点和输出节点之间，既是输出又是输入`
## REDIS_READER
`输入节点-读 redis`
## REDIS_WRITER
`输出节点-写 redis`
## CUSTOM_READER_WRITER
`自定义节点，通过嵌入go脚本来实现各种操作`
## EXECUTE_SHELL
`输入节点-执行系统脚本节点`
## CSV_READER
`输入节点-读取CSV文件节点`

## 组合方式
- `DB_INPUT_TABLE -> DB_OUT_TABLE `
- `DB_INPUT_TABLE -> XLS_WRITER `
- `DB_INPUT_TABLE -> MQ_PRODUCER `
- `DB_INPUT_TABLE -> REDIS_WRITER `
- `XLS_READER -> DB_OUT_TABLE `
- `XLS_READER -> XLS_WRITER `
- `XLS_READER -> MQ_PRODUCER `
- `XLS_READER -> REDIS_WRITER `
- `DB_EXECUTE_TABLE -> OUTPUT_TRASH `
- `DB_EXECUTE_TABLE -> DB_OUT_TABLE `
- `DB_EXECUTE_TABLE -> XLS_WRITER`
- `DB_EXECUTE_TABLE -> MQ_PRODUCER`
- `DB_EXECUTE_TABLE -> REDIS_WRITER`
- `MQ_CONSUMER -> DB_OUT_TABLE`
- `MQ_CONSUMER -> XLS_WRITER`
- `MQ_CONSUMER -> MQ_PRODUCER`
- `MQ_CONSUMER -> REDIS_WRITER`
- `DB_INPUT_TABLE -> COPY_STREAM `
- `XLS_READER -> COPY_STREAM `
- `MQ_CONSUMER -> COPY_STREAM `
- `REDIS_READER -> COPY_STREAM `
- `COPY_STREAM -> DB_OUT_TABLE `
- `COPY_STREAM -> XLS_WRITER `
- `COPY_STREAM -> MQ_PRODUCER `
- `COPY_STREAM -> REDIS_WRITE `
- `REDIS_READER -> DB_OUT_TABLE `
- `REDIS_READER -> XLS_WRITER `
- `REDIS_READER -> MQ_PRODUCER `
- `REDIS_READER -> REDIS_WRITER `
- `CUSTOM_READER_WRITER -> OUTPUT_TRASH `
- `EXECUTE_SHELL -> DB_OUT_TABLE `
- `EXECUTE_SHELL -> XLS_WRITER `
- `EXECUTE_SHELL -> MQ_PRODUCER `
- `EXECUTE_SHELL -> REDIS_WRITER `
- `EXECUTE_SHELL -> OUTPUT_TRASH `
- `CSV_READER -> DB_OUT_TABLE `
- `CSV_READER -> XLS_WRITER `
- `CSV_READER -> MQ_PRODUCER `
- `CSV_READER -> REDIS_WRITER `
- `CSV_READER -> OUTPUT_TRASH `


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
MYSQL、Influxdb 1x、CK、sqlite

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
|script| insert、delete、update SQL语句 |ck,mysql,sqlite|
| batchSize       | 每次批提交的记录数                  | ck,mysql,sqlite <br/>注意influx以输入时的fetchSize为批提交的大小 |
| outputFields    | 输入节点读数据时传递过来的字段名称          | influx,ck,mysql,sqlite                         |
| renameOutputFields    | 输出节点到目标数据源的字段名称            | influx,ck,mysql,sqlite                         |
| dbConnection | 数据源ID                      ||
| desc         | 描述                         ||
|outputTags| 输入节点读数据时传递过来的标签名称          | influx                                  |
|renameOutputTags| 输出节点到目标数据源的标签名称            | influx                                  |
|rp| 保留策略名称                     | influx                                  |
|measurement| 表名称                        | influx                                  |

## 支持目标类型
MYSQL、Influxdb 1x、CK、sqlite


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
| sqlScript | delete、update语句，多条语句之间用分号分隔 | mysql，sqlite，ck(不支持delete,update)     |
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

### mq消息者 （支持kafka）

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
| type   | 数据源类型    |INFLUXDB_V1、MYSQL、CLICKHOUSE、SQLITE|
| dbURL | 连接地址     | ck,mysql,influx    |
| database   | 数据库名称    | ck,mysql,influx,sqlite    |
| username   | 用户名称     | ck,mysql,influx    |
| password   | 密码       | ck,mysql,influx    |
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
run_graph -fileUrl ./global6.xml -logLevel debug arg1="d:/test3.xlsx" arg2=上海
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

# 支持解析嵌入go语言
可以在任意一个输出节点的 `<BeforeOut></BeforeOut>` 标签内嵌入自己的业务逻辑

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


