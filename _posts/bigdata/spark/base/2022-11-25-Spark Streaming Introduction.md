---
title: Spark Streaming Introduction
author: Lynn
date: 2022-11-25 22:45:00 +0800
last_modified_at: 2023-01-30 22:45:00 +0800
categories: [Bigdata, Spark]
tags: [spark]
---
## Spark Streaming基本介绍
### Spark Streaming
Spark Streaming 用于流式数据的处理。Spark Streaming 支持的数据输入源很多，例如：Kafka、 Flume、Twitter、ZeroMQ 和简单的 TCP 套接字等等。数据输入后可以用 Spark 的高度抽象原语,如：map、reduce、join、window 等进行运算。而结果也能保存在很多地方，如 HDFS，数据库等。

### Spark Streaming数据结构
#### DStream
- 和 Spark 基于 RDD 的概念很相似，Spark Streaming 使用离散化流(discretized stream)作为抽象表示，叫作 DStream。
- DStream 是随时间推移而收到的数据的序列。在内部，每个时间区间收到的数据都作为 RDD 存在，而 DStream 是由这些 RDD 所组成的序列(因此得名“离散化”)。所以简单来讲，DStream 就是对 RDD 在实时数据处理场景的一种封装。