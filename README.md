# etl-engine
Data exchange
实现从influxdb v1读取数据 -> 目标数据源数据类型转换 -> 写到目标数据源 （目前源数据源支持 influxdb v1、ck、mysql、excel，目标数据源支持 influxdb v1、ck、mysql、excel）
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
  <Line id="LINE_01" type="LINE_1" from="DB_INPUT_01" to="DB_OUTPUT_01" order="0" metadata="METADATA_01"></Line>
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
## 组合方式
- `DB_INPUT_TABLE -> DB_OUT_TABLE `
- `DB_INPUT_TABLE -> XLS_WRITER `
- `XLS_READER -> DB_OUT_TABLE `
- `XLS_READER -> XLS_WRITER `
- `DB_EXECUTE_TABLE -> OUTPUT_TRASH `
- `DB_EXECUTE_TABLE -> DB_OUT_TABLE `
- `DB_EXECUTE_TABLE -> XLS_WRITER`

# 支持配置全局变量
- ### 通过命令行方式传递全局变量

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

