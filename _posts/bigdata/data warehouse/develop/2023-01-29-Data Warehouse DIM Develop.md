---
title: Data Warehouse DIM Develop
author: Lynn
date: 2023-01-27 10:30:00 +0800
last_modified_at: 2023-01-27 10:35:00 +0800
categories: [Bigdata, DataWarehouse]
tags: [data warehouse]
syntax: colorful
---
## DIM层
### DIM层设计要点
- DIM层的设计一句是维度建模理论，该层存储维度模型的维度表（重点）
- DIM层的数据存储格式为orc列式存储+snappy压缩（相较于ODS的存储格式和压缩更快）
  + ODS在乎的是减少存储空间，其他层在乎的是查询更快
- DIM层表名的命名规范为dim_表名_全量表或者拉链表标识（full/zip）
### 维度表设计：
- 确定维度表
- 确定主维度表和相关维度表
- 确定维度属性
  + 尽可能生成丰富的维度属性 
  + 尽量不使用编码而使用文字说明或者编码加文字说明 
  + 沉淀出通用的维度属性 
  + 商品维度表
    * 可以参考原始数据在MySQL中的表，找到商品MySQL表并找到与其相关联的其他MySQL表，考虑是否和维度表有关，在考虑相关的Hive维度表如何设计。
    * 全量表数据，数据装载从ODS的一个分区到DIM层的一个分区，时间分区要对应上

## DIM层 dev experience
### Hive复杂数据类型构建
+ **array**：
  * array() 一行一出: array(val1,val2…)
  * collect_set() 多行一出

+ **map**:
  * map():map(key1,val1,key2,val2…)
  * str_to_map(text,delimiter1,delimiter2):
    * delimiter1用于分割k-v键值对，默认值为 ',' 
    * delimiter2用户分割k和v,默认值为':'

+ **struct**:
  * struct(val1,val2,val3…):values为表的column
  * named_struct(name1,val1,name2,val2…):同时给出field的name和val

### CTE语法
common table expression公共表表达式
+ 原来一个表自己join自己需要写出两遍一样的表，提前声明子查询的表只写一遍供其他公共部分使用
语法
  * with table_name as(select_expression) 这就声明好了一个子查询并写出，可以重复引用这个table_name
  * 想要一次声明多个不同的子查询，语法如下: 其中with只需要写一遍
```hql
    WITH
        table_name1 as(select_expression1),
        table_name2 as(select_expression2),
        table_namen as(select_expressionn)
    SELECT xxx
```

## DIM层 dev question
1. 表信息保存
   * 表中数据需要以orc文件形式保存，而原始数据只有txt文件或者其他类型的文件，可以先创建一个tmp的临时表（默认指定为TEXTFILE存储格式，或者指定成其他对应类型的文件），然后通过insert语句将临时表的数据写入到最终的成表内部解决因为表数据存储格式不同带来的读取问题。
   * 例子：数据保存在txt文件当中，但是表指定stored as orc，如果直接将txt存储到表路径下，最后读取的数据会因为文件格式出错，此时可以创建一个tmp临时表，将txt数据保存到临时表路径下，然后通过insert overwrite into table select * from tmp_table，hive底层会将存储格式进行转化将文件转存为orc格式 
     * 原理：Hive底层建表时都会解析成对应的InputFormat和OutputFormat，利用TextInputFormat从临时数据表读取数据，并通过OrcOutputFormat写入到指定表中完成文件格式的转换 
     * 如何查看一个表的具体信息：
```hql
    SHOW CREATE TABLE table_name 
```
     * 可以查看一个表的各种具体信息，包括各类InputFormat，OutputFormat
2. 处理敏感信息
   * 表中存在部分敏感信息，比如用户具体名称和联系方式等数据，可以在数据装载的时候使用Hive内部内置的加密函数进行脱敏处理:md5(data)
3. Hive动态分区
   * 通过数据自身特性通过一个insert语句将不同的数据写到不同的分区内
   * 将所有结果集union合并以下（不同行），然后利用动态分区将数据写到不同的分区内 
4. 处理拉链表类型的增量表数据
   * 首日和其他时间的处理逻辑是不一样的 
     * 首日：全量表为最初始的用户表，初始的拉链表就是最开始的全量表 
     * 其他时间：根据表的操作时间（binlog等方式）得到用户的变化表，然后再将变化表与之前的拉链表进行合并得到新的拉链表 
       * 合并：取出之前拉链表中没有变化的数据，变化的数据取出放入过期分区后将变化后的数据和变化新增的数据与没有变化的数据合并成新的分区表 
   * 具体操作SQL流程如下： 
     1. 将昨日的旧表取出，并于今日新增的表进行全外连接，这个操作会在两个表某一字段（一般是所谓的主码概念）上进行，分为以下几种情况 
       * 旧表有的数据，而新表没有，将这部分旧表有的数据直接保留下来 
       * 旧表没有的数据，而新表有的数据，将这部分新的数据直接追加到表中 
       * 旧表和新表都有的数据，说明发生了update操作，需要根据具体的业务逻辑确认需要保留新旧表的哪些字段。 
     2. 将上一步中三种情况获取的全外连接表保留下来，供下一步insert装载数据使用，可以使用CTE语法保留这张表。 
     3. 使用动态分区的手段，通过一条insert将数据分隔开，比如如果通过时间进行分区，可以在insert代码后加上partition(时间)，时间这个字段必须在全外连接的表中存在。
