---
title: Spark Data Structure Introduction 
author: Lynn 
date: 2022-11-25 22:30:00 +0800 
last_modified_at: 2023-01-31 18:35:00 +0800 
categories: [Bigdata, Spark]
tags: [spark]
---

## 核心编程：

Spark 计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是:

- 累加器：分布式共享只写变量
- 广播变量：分布式共享只读变量
- RDD : 弹性分布式数据集

## 累加器:

累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在 Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，传回 Driver 端进行
merge。

- merge的原因: spark的分布式计算，会将数据分到不同的Executor上计算完
![img.png](/blog_imgs/spark/base/spark data structure introduction/img0.png)
- 分布式共享的只写变量
    + 分布式：累加器的值是分布式统计执行的
    + 共享：累加器被多个Executor共享
    + 只写：说明Executor之间累加器是不同互相访问，是读不到的，只有Driver可以读取每个Executor上的累加器
- 累加器中会出现少加和多加的情况
    + 少加：转换算子中调用了累加器，但是没有执行行动算子，那么累加器不会执行生效
    + 多家：多次调用了行动算子，累加器会多次执行
    + 为了防止少加和多加的情况，一般会把累加器放在行动算子中使用，而不会放在转换算子中

### 累加器类别
- 系统累加器：
    + 通过sparkContext获取
  ```scala
  var acc1 = sc.longAccumulator("acc_name")
  sc.doubleAccumulator("acc_name")
  sc.collectionAccumulator("acc_name")
  ```

    + 使用累加器实例代码:
  ```scala
  rdd.foreach(num=>{acc1.sum(num)})
  ```

- 自定义累加器：
    1. 继承AccumulatorV2接口，定义泛型，定义IN累加器输入的数据类型，定义OUT累加器输出的类型
       +  class MyAccumulator extends AccumulatorV2[IN,OUT]
    2. 重写实现累加器接口中的方法: add 
    3. 向sc注册累加器 
    ```scala
    val acc = new MyAccumulator()
    sc.register(acc,"acc_name")
    ```
  
    4. 使用累加器打印结果
    ```scala
    rdd.foreach(num=>{acc.add(num)})
    print(acc.value)
    ```

## 广播变量:

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表， 广播变量用起来都很顺手。在多个并行操作中使用同一个变量，Spark
会为每个任务分别发送（闭包数据都是以Task为单位发送的），会导致在Executor中包含同样的一份数据导致冗余，占用大量的内存。广播变量就是将数据放在Executor的内存当中，而不是放在每一个task中，从而达到共享的目的。这份数据只读，保证线程安全。
- 分布式共享只读变量
- 声明广播变量
```scala
  val broadcast = sc.broadcast(list)
```

## RDD:

全称为 resilient distributed dataset 弹性分布式数据集, Spark中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。每个RDD都是一个计算单元，将多个RDD进行关联组成一个完整的需求计算逻辑。

### 弹性分布式数据集
- 弹性
  + 存储的弹性：内存与磁盘的自动切换；
  + 容错的弹性：数据丢失可以自动恢复；
  + 计算的弹性：计算出错重试机制；
  + 分片的弹性：可根据需要重新分片。
- 分布式：数据存储在大数据集群不同节点上
- 数据集：RDD 封装了计算逻辑，并不保存数据
- 数据抽象：RDD 是一个抽象类，需要子类具体实现
- 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的 RDD，在新的 RDD 里面封装计算逻辑
- 可分区、并行计算 

### 核心属性
- 分区列表:list of partitions
  + RDD数据结构中存在分区列表，用于执行任务时并行计算，是实现分布式计算的重要属性
- 分区计算函数:function for computing each split 
  + compute() 
  + Spark计算的时候使用分区函数对每一个分区进行计算 
  + 一个分区都会有一个task，但是所有分区的计算逻辑和方式相同 
- RDD之间的依赖关系:dependencies on each RDD 
  + getDependencies() 
  + RDD是计算模型的封装，当需求当中需要将多个计算模型进行组合时，需要将多个RDD建立依赖关系 
- 分区器(可选):a partitioner for key-value RDDs 
  + partitioner:Option[Partitioner] 
  + 将数据进行分区处理，逻辑可以自己定义（要求数据为KV类型数据） 
- 首选位置(可选 ):preferred locations 计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算（比如选择节点距离最近的服务器来运行计算服务等） 
  + getPreferredLocations 
  + 判断计算发送哪个节点效率最优，移动数据不如移动计算