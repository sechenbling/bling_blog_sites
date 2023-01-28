---
title: Data Warehouse DWD Develop
author: Lynn
date: 2023-01-27 11:30:00 +0800
last_modified_at: 2023-01-27 11:35:00 +0800
categories: [Bigdata, DataWarehouse]
tags: [data warehouse]
syntax: colorful
---

## DWD层

### DWD层设计要点
- DWD层的设计依据是维度建模理论，该层存储维度模型的维度表（重点）
- DWD层的数据存储格式为orc列式存储+snappy压缩
  + DWD层表名的命名规范为dwd_数据域_表名_单分区全量增量标识（full/inc）
  + 数据域：对数据进行纵向的分类，方便数据管理和使用，每一类称为一个域，通常是按照业务过程来划分数据域，业务过程其实就是事实表，其实就是对事实表进行划分。

### 事实表设计：
- 根据业务总线矩阵（确认业务，声明粒度，确认维度，确认事实），明确业务过程和哪个维度有关系并不是需求决定的，而是根据数仓的具体业务逻辑决定的，维度的确定是比较灵活的，尽可能多的确定与业务过程相关的维度即可。 
- 增量分区都是每天一个分区，每个分区存放当天新增的数据（增量表逻辑相同） 
- 首日假设根据之前数据都是一次操作，根据数据中的字段进行按照数据内的字段动态分区 
- 后续的数据按照具体当天的时间分区放即可 
- 每个数据域的事实表总共写两个SQL

### Hive时间戳转换
- from_unixtime(bigint,time_format)，固定从unix epoch(1970-01-01)零时区计时 
- from_utc_timestamp(ts,时区):
  + 东八区为'GMT+08'
  + ts为double时单位为秒，为int时单位为毫秒，要注意类型看是否需要进行单位转换 
  + 这个函数只是转换ts的数值，如果想要输出yyyy-MM-dd的格式，需要在外层套date_format(from_utc_timestamp,time_format)
