---
title: HDFS NameNode&SecondaryNameNode
author: Lynn
date: 2024-02-01 22:33:00 +0800
last_modified_at: 2024-02-01 22:33:00 +0800
categories: [Bigdata, Hadoop]
tags: [hdfs]
syntax: colorful
---

* fslmage 镜像文件：存储数据
HDFS文件系统元数据的一个永久性的检查点，其中包含HDFS文件系统的所有目录和文件inode的序列化信息。

* Edits 记录变化的步骤
存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先会被记录到Edits文件当中
  * seen_txid:记录当前最新的Edits(即最后一个Edits的数字 
  * VERSION：记录集群ID

每次NameNode启动的时候都会将Fsimage文件读入内存，加载Edits的更新操作，保证数据是最新的同步的，可以看作NameNode启动的时候就将Fsimage和Edits文件进行了合并

* 上线后内存的中出现的是fslmage和执行完Edits后的结果
![img.png](/blog_imgs/hadoop/hdfs/img_1.png)
    * 差别在inprogress这个文件 2NN在更新的时候如果有新的操作，会记录在inprogress文件当中 
    * NN 和 2NN 工作机制 思考：NameNode 中的元数据是存储在哪里的？ 首先，我们做个假设，如果存储在 NameNode 节点的磁盘中，因为经常需要进行随机访 问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在 内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的 FsImage。 这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新 FsImage，就会导 致效率过低，但如果不更新，就会发生一致性问题，一旦 NameNode 节点断电，就会产生数 据丢失。因此，引入 Edits 文件（只进行追加操作，效率很高）。每当元数据有更新或者添 加元数据时，修改内存中的元数据并追加到 Edits 中。这样，一旦 NameNode 节点断电，可 以通过 FsImage 和 Edits 的合并，合成元数据。 但是，如果长时间添加数据到 Edits 中，会导致该文件数据过大，效率降低，而且一旦 断电，恢复元数据需要的时间过长。因此，需要定期进行 FsImage 和 Edits 的合并，如果这 个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode， 专门用于 FsImage 和 Edits 的合并。 

* 第一阶段：NameNode 启动 
第一次启动:
  1. NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。 
  2. 客户端对元数据进行增删改的请求。 
  3. NameNode 记录操作日志，更新滚动日志。 
  4. NameNode 在内存中对元数据进行增删改。
* 第二阶段：Secondary NameNode 工作 
  1. Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode 是否检查结果。
  2. Secondary NameNode 请求执行 CheckPoint。
  3. NameNode 滚动正在写的 Edits 日志。
  4. 将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。
  5. Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。 
  6. 生成新的镜像文件 fsimage.chkpoint。 
  7. 拷贝 fsimage.chkpoint 到 NameNode。 
  8. NameNode 将 fsimage.chkpoint 重新命名成 fsimage。

