---
title: Data Warehouse ODS Develop
author: Lynn
date: 2023-01-27 09:30:00 +0800
last_modified_at: 2023-01-27 09:35:00 +0800
categories: [Bigdata, DataWarehouse]
tags: [data warehouse]
syntax: colorful
---

## ODS层
- 表结构设计依托于从业务系统同步过来的数据结构
- ODS层要保存全部的历史数据，数据量较大，使用压缩程度较高的格式gzip
- ODS层的命名规范为ods_表名_单分区增量全量标识(inc或者full)

## Hive数仓启动
- 启动Hive服务
```shell
  nohup ./hive --service metastore &
  nohup ./hive --service hiveserver2 &
```
- HiveServer的相关问题
  + OOM问题（导致连接断开或者SQL语句无法运行）：修改conf/hive-env.sh中的HADOOP_HEAPSIZE大小
- Hive相关问题：
  + MySQL中的空值定义（null）与Hive的空值定义(\N)不一样，且DataX并没有提供将MySQL的null值转化为空值的功能，为了保证同步的数据能够正常被Hive解析，可以在建表的时候指定将空值存为空字符串 NULL DEFINED AS''。但是反过来从HDFS导到MySQL可以自定义空值形式。 

## ODS层表
- 建表：
  + 日志表：分为启动日志和具体操作日志，可以将所有日志的字段组合在一起，一条日志不存在的字段的数据可以设置为空，比如启动日志中含有'start'，却没有操作日志中的'actions',可以将两个字段都放在同一个日志表当中。
  + 全量表：按照时间分区，每一天全部的数据都会同步过来。
  + 增量表：按照时间分区，增量表中每一行的数据为一次变更操作的记录，包括操作类型（insert update），变化后的数据（data字段）和原来旧的数据是什么（old字段）等。
- JSON表建立（Hive中）：
  + 语法：
```hql
  CREATE TABLE table_name(FIELDS VARCHAR(10))
  ROW FROMAT SERDE 'org.apache.hadoop.hive.serde2.JsonSerDe'
  STORE AS TEXTFILE;
``` 
  + SERDE为序列化器和反序列化器的缩写 
    * 在Hive中Hive使用SerDe来读和写表的行
      * HDFS files -> InputFileFormat -> <key,value> -> Deserializer -> RowObject
      * RowObject-> Serializer-> <key,value>-> OutputFileFormat -> HDFS files
    * Hive建表底层都会解析成对应的InputFormat，OutputFormat，SerDe
  + STORE AS存储格式 默认为TEXTFILE
  + 表内的字段名要与JSON的一级键名对应上才能获取对应字段的数据，但是对应不上也不会报错~
    * 如果存在嵌套结构的话，可以使用Hive内置的其他复杂类型实现，比如
      * STRUCT<name1:type1,name2:type2,name3:type3…>:sturct.name 取值
      * Array<Type>:通过arr[index] 取值
      * Map<keyType,valueType>:通过map['key'] 取值
    * 复杂数据类型也是可以相互嵌套的，嵌套的字段名也要和键名对应
  + 在Hive中gzip和bzip2压缩的文本文件可以直接作为TEXTFILE被Hive解析，其他压缩格式可以参考官网文档选用对应压缩格式的建表语句。
  + 项目中建立的都是外部表 CREATE EXTERNAL TABLE;
    * 内部表由Hive自己管理，而外部表由HDFS管理
    * 删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS上的文件并不会被删除；
	
	
