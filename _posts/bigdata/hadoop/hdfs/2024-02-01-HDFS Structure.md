---
title: Hadoop HDFS Structure
author: Lynn
date: 2024-02-01 22:31:00 +0800
last_modified_at: 2024-02-01 22:31:00 +0800
categories: [Bigdata, Hadoop]
tags: [hdfs]
syntax: colorful
---

## NameNode：master 主管管理者
* 管理HDFS的名称空间（原数据） 
* 配置副本策略 
  * 设置不同数据的副本的数量 
* 管理数据块的映射信息 
  * 比较大的文件分块存储，而且块和块之间的存储没有关系 
* 处理客户端的读写请求

## DataNode 
就是slave worker，NameNode下达命令由DataNode执行实际的操作
* 存储实际的数据块 
* 执行数据块的读写操作

![img.png](/blog_imgs/hadoop/hdfs/img.png)

## SecondaryNameNode 2NN
* 并非NameNode的热备，当NN挂掉的时候它并不能马上替换NN提供服务
* 辅助NN分担工作：比如定期合并镜像文件和编辑日志(Fsimage&Edits)并推送给NN
* 紧急情况下可以恢复部分数据（不是所有数据都给了2NN）

## Client 客户端
* 文件切分：文件上传HDFS时，Client将文件切分成一个一个的Block后上传
* 与NN交互获取文件位置
* 与DN交互读写数据
* 提供一些命令来管理HDFS：比如NN格式化
* 通过一些命令来访问HDFS：进行增删改查

## 文件块大小：
只是指占用上限，如果一个文件小于128M其占用磁盘就是其本身
* Hadoop2，3是128M
* Hadoop1是64M
* 找到块的时间是传输块时间的1%最佳
* 块大小设置
  * 块太小，会增加寻址时间，程序一直在找块的开始位置（块数量多了）
  * 块太大，从磁盘的传输时间会明显大于定位这个块开始位置所需的时间，在处理这块数据时会非常慢
  * 块大小设置:主要取决于磁盘传输速率
			
