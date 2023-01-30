---
title: Spark Sql Introduction
author: Lynn
date: 2022-11-25 22:40:00 +0800
last_modified_at: 2023-01-30 22:40:00 +0800
categories: [Bigdata, Spark]
tags: [spark]
---

## SparkSQL基本介绍
### Spark Sql
Spark SQL 是Spark用于结构化数据处理的Spark模块:
- 数据兼容方面: SparkSQL 不但兼容 Hive，还可以从 RDD、parquet 文件、JSON 文件中获取数据，未来版本甚至支持获取 RDBMS 数据以及 cassandra 等 NOSQL 数据； 
- 性能优化方面: 除了采取 In-Memory Columnar Storage、byte-code generation 等优化技术外、将会引进 Cost Model 对查询进行动态评估、获取最佳物理计划等等；
- 组件扩展方面: 无论是 SQL 的语法解析器、分析器还是优化器都可以重新定义，进行扩展。

### Spark Sql数据结构
#### DataSet
  - 在 Spark 中，DataFrame 是一种以 RDD 为基础的分布式数据集，类似于传统数据库中的二维表格。DataFrame 与 RDD 的主要区别在于，前者带有 schema 元信息，即 DataFrame 所表示的二维表数据集的每一列都带有名称和类型。这使得 Spark SQL 得以洞察更多的结构信息，从而对藏于 DataFrame 背后的数据源以及作用于 DataFrame 之上的变换进行了针对性 的优化，最终达到大幅提升运行时效率的目标。反观 RDD，由于无从得知所存数据元素的具体内部结构，Spark Core 只能在 stage 层面进行简单、通用的流水线优化。 
  - 与 Hive 类似，DataFrame 也支持嵌套数据类型（struct、array 和 map）。从 API 易用性的角度上看，DataFrame API 提供的是一套高层的关系操作，比函数式的 RDD API 要更加友好，门槛更低 
  - DataFrame也是懒执行的（就是将某些逻辑延迟到使用时再计算），但是性能比RDD更高，主要原因是执行计划优化，即查询计划通过 Spark catalyst optimiser 进行优化。简而言之， 逻辑查询计划优化就是一个利用基于关系代数的等价变换，将高成本的操作替换为低成本操作的过程。

#### DataFrame
  - DataSet是分布式数据集合，DataSet 是 Spark 1.6 中添加的一个新抽象，是 DataFrame 的一个扩展。它提供了 RDD 的优势（强类型，使用强大的 lambda 函数的能力）以及 Spark SQL 优化执行引擎的优点。DataSet 也可以使用功能性的转换（操作 map，flatMap，filter 等等）。 
  - DataSet 是 DataFrame API 的一个扩展，是 SparkSQL 最新的数据抽象 
  - 用户友好的 API 风格，既具有类型安全检查也具有 DataFrame 的查询优化特性； 
  - 用样例类来对 DataSet 中定义数据的结构信息，样例类中每个属性的名称直接映射到 DataSet 中的字段名称； 
  - DataSet 是强类型的。比如可以有 DataSet[Car]，DataSet[Person]。 
  - DataFrame 是 DataSet 的特例，DataFrame=DataSet[Row] ，所以可以通过 as 方法将 DataFrame 转换为 DataSet。Row 是一个类型，跟Car、Person 这些的类型一样，所有的表结构信息都用 Row 来表示。获取数据时需要指定顺序。

### Spark Sql特点
  - 易整合：整合SQL查询和Spark编程
  - 统一的数据访问：使用相同的方式连接不同的数据源
  - 兼容Hive：可以在已有的仓库上运行SQL或者HiveSQL
  - 标准数据连接：通过JDBC/ODBC连接