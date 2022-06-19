# etl-engine
Data exchange
实现从源读取数据 -> 目标数据类型转换 -> 写到目标数据源 （目前源数据支持 influxdb v1、ck、mysql、excel、mq，目标数据支持 influxdb v1、ck、mysql、excel、mq）

`数据中台必备，架构师利器`


[下载地址](https://github.com/hw2499/etl-engine/releases/tag/v1.0.0)

# 数据流图

![输入输出](https://raw.githubusercontent.com/hw2499/etl-engine/main/docs/input_output.png)

![数据流复制](https://raw.githubusercontent.com/hw2499/etl-engine/main/docs/copy_stream.png)



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
  <Line id="LINE_01" type="STANDARD" from="DB_INPUT_01" to="DB_OUTPUT_01" order="0" metadata="METADATA_01"></Line>
  <Metadata id="METADATA_01">
    <Field name="c1" type="string" default="-1" nullable="false"/>
    <Field name="c2" type="int" default="-1" nullable="false"/>
    <Field name="tag_1" type="string" default="-1" nullable="false"/>
    <Field name="tag_4" type="string" default="-1" nullable="false"/>
  </Metadata>
  <Connection id="CONNECT_01" dbURL="http://127.0.0.1:58080" database="db1" username="user1" password="******" token=" " org="hw"  type="INFLUXDB_V1"/>

  <Connection id="CONNECT_02" dbURL="http://127.0.0.1:58086" database="db1" username="user1" password="******" token=" " org="hw"  type="INFLUXDB_V1"/>
 <!--    <Connection id="CONNECT_02" dbURL="127.0.0.1:19000" database="db1" username="user1" password="******" type="CLICKHOUSE"/>-->
  <!--    <Connection id="CONNECT_02" dbURL="127.0.0.1:3306" database="db1" username="user1" password="******" type="MYSQL"/>-->
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


## 组合方式
- `DB_INPUT_TABLE -> DB_OUT_TABLE `
- `DB_INPUT_TABLE -> XLS_WRITER `
- `DB_INPUT_TABLE -> MQ_PRODUCER `
- `XLS_READER -> DB_OUT_TABLE `
- `XLS_READER -> XLS_WRITER `
- `XLS_READER -> MQ_PRODUCER `
- `DB_EXECUTE_TABLE -> OUTPUT_TRASH `
- `DB_EXECUTE_TABLE -> DB_OUT_TABLE `
- `DB_EXECUTE_TABLE -> XLS_WRITER`
- `DB_EXECUTE_TABLE -> MQ_PRODUCER`
- `MQ_CONSUMER -> DB_OUT_TABLE`
- `MQ_CONSUMER -> XLS_WRITER`
- `MQ_CONSUMER -> MQ_PRODUCER`
- `DB_INPUT_TABLE -> COPY_STREAM `
- `XLS_READER -> COPY_STREAM `
- `MQ_CONSUMER -> COPY_STREAM `
- `COPY_STREAM -> DB_OUT_TABLE `
- `COPY_STREAM -> XLS_WRITER `
- `COPY_STREAM -> MQ_PRODUCER `


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
MYSQL、Influxdb 1x、CK

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
| startRow    | 从第几行开始读取                                                      |
| sheetName | 表名称                                                           |
| fieldMap         | 字段映射关系，格式：field1=A;field2=B;field3=C<br/>字段名称=第几列 多个字段之间用分号分隔 |

### 样本
```shell
  <Node id="XLS_READER_01"   type="XLS_READER" desc="输入节点1"  fileURL="d:/demo/test1.xlsx" startRow="2" sheetName="人员信息" fieldMap="field1=1;field2=2;field3=3">
  </Node>
```

## 节点DB_OUTPUT_TABLE
`输出节点`

| 属性           | 说明                         | 适合                                      |
|--------------|----------------------------|-----------------------------------------|
| id          | 唯一标示                       ||
| type         | 类型, DB_OUTPUT_TABLE        ||
|sqlScript| insert、delete、update SQL语句 |ck,mysql|
| batchSize       | 每次批提交的记录数                  | ck,mysql <br/>注意influx以输入时的fetchSize为批提交的大小 |
| outputFields    | 输入节点读数据时传递过来的字段名称          | influx,ck,mysql                         |
| renameOutputFields    | 输出节点到目标数据源的字段名称            | influx,ck,mysql                         |
| dbConnection | 数据源ID                      ||
| desc         | 描述                         ||
|outputTags| 输入节点读数据时传递过来的标签名称          | influx                                  |
|renameOutputTags| 输出节点到目标数据源的标签名称            | influx                                  |
|rp| 保留策略名称                     | influx                                  |
|measurement| 表名称                        | influx                                  |

## 支持目标类型
MYSQL、Influxdb 1x、CK


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
| type         | 类型, XLS_WRITER                                    |
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


## 节点MQ_CONSUMER
`输入节点`
### mq消费者 （目前支持rocketmq、kafka）

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

## 节点MQ_PRODUCER
`输出节点`
### mq生产者 （目前支持rocketmq、kafka）

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
可以嵌入自己的业务逻辑
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
		area := gjson.Get(tmpStr,"tag_1").String()
		year := gjson.Get(tmpStr,"c3").String()
		tmpStr, _ = sjson.Set(tmpStr, "tag_1", area + "_" + year)
		newRows, _ = sjson.SetRaw(newRows, "rows.-1", tmpStr)
	}
	return newRows, nil
}

```

# 合作模式
<details><summary>欢迎对接合作</summary>
 

#### etl-engine 寻求合作模式中...

    ```
     @auth Mr Huang
     vx:weigeonlyyou
    ```

</details> 


