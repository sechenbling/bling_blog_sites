---
title: Read&Write Process
author: Lynn
date: 2024-02-01 22:33:00 +0800
last_modified_at: 2024-02-01 22:33:00 +0800
categories: [Bigdata, Hadoop]
tags: [hdfs]
syntax: colorful
---

## 写入数据的流程：
1. 客户端创建分布式文件系统DistributedFileSystem 
2. 向NameNode请求上传文件
   * NameNode检查权限
   * NameNode检查目录结构（目录是否存在） 
3. NameNode响应可以上传文件 
4. 客户端请求上传第一个Block(0-128M)，请求NameNode返回DataNode 
5. NameNode返回多个DataNode表示这三个节点存储数据
   * 选择策略(节点距离最近，负载均衡)： 
       - 本地节点
       - 其他机架的一个节点
       - 其他机架的另一个节点 
6. 客户端创建FSDataOutputStream上传数据，向一个DN请求建立Block传输通道，之后再由DN向其他的DN请求建立通道（DN边存储边传输给其他DN），等数据传输完毕进行应答
   * 流的传输单位为packet(64K)
       - packet由512B(chunk)+4B(校验位)的chunk累计到64K后形成的
       - ack队列来接收应答

![img.png](/blog_imgs/hadoop/hdfs/img_2.png)

#### 网络拓扑-节点距离计算
节点之间到共同祖先的距离之和为节点的距离

![img.png](/blog_imgs/hadoop/hdfs/img_3.png)

#### 机架感知：副本存储节点的选择
假如由三个副本，其中一个副本存储在客户端所在集群 节点上，如果客户端在集群外，则随机选择一个；其他两个副本：一个在另一个机架的随机一个节点，一个在第二个副本所在机架的随机节点。

![img.png](/blog_imgs/hadoop/hdfs/img_4.png)

## 读取数据的流程
1. 客户端创建分布式文件系统对象向NameNode请求下载数据 
2. NameNode判断权限和文件是否存在，如果都满足就将目标文件的元数据信息反馈回去 
3. 客户端创建FSDataInputStream选择节点距离最近的节点(首要原则)进行读取（此外还会考虑节点的负载均衡（次要原则） 
   * 如果还有其他的块，但当前节点的负债能力已经达到极限了就选择其他的节点进行读取 
   * 读取是串行读，先读取第一块，再读取第二块进行拼接
