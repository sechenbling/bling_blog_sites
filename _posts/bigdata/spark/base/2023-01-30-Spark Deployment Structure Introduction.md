---
title: Spark Deployment Structure Introduction
author: Lynn
date: 2023-01-30 22:30:00 +0800
last_modified_at: 2023-01-30 22:30:00 +0800
categories: [Bigdata, Spark]
tags: [spark]
---
## Spark Introduction
### Spark Summary
1. Spark 是一种由 Scala 语言开发的快速、通用、可扩展的大数据分析引擎 
2. Spark Core 中提供了 Spark 最基础与最核心的功能 
3. Spark SQL 是 Spark 用来操作结构化数据的组件。通过 Spark SQL，用户可以使用 SQL 或者 Apache Hive 版本的 SQL 方言（HQL）来查询数据。 
4. Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件，提供了丰富的 处理数据流的 API。 
5. Spark 出现的时间相对较晚，并且主要功能主要是用于数据计算， 所以其实 Spark 一直被认为是 Hadoop 框架的升级版。

### Spark on Hadoop
- Spark 就是在传统的 MapReduce 计算框架的基础上，利用其计算过程的优化，从而大大加快了数据分析、挖掘的运行和读写速度，并将计算单元缩小到更适合并行计算和重复使用的 RDD 计算模型。
- Spark 和Hadoop 的根本差异是多个作业之间的数据通信问题 : Spark 多个作业之间数据 通信是基于内存，而 Hadoop 是基于磁盘。 
- Spark Task 的启动时间快。Spark 采用 fork 线程的方式，而 Hadoop 采用创建新的进程 的方式。 
- Spark 只有在shuffle 的时候将数据写入磁盘，而 Hadoop 中多个 MR 作业之间的数据交 互都要依赖于磁盘交互 
- Spark 的缓存机制比 HDFS 的缓存机制高效。 
- 但是 Spark 是基于内存的，所以在实际的生产环境中，由于内存的限制，可能会 由于内存资源不够导致 Job 执行失败，此时，MapReduce 其实是一个更好的选择，所以 Spark 并不能完全替代 MR。

### Spark Core 
- Spark Core 中提供了 Spark 最基础与最核心的功能，Spark 其他的功能如：Spark SQL， Spark Streaming，GraphX, MLlib 都是在 Spark Core的基础上进行扩展的 
- Spark SQL：Spark SQL 是 Spark 用来操作结构化数据的组件。通过 Spark SQL，用户可以使用 SQL 或者 Apache Hive 版本的 SQL 方言（HQL）来查询数据。
- Spark Streaming：Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件，提供了丰富的处理 数据流的 API。
- Spark MLlib：MLlib 是 Spark 提供的一个机器学习算法库。MLlib 不仅提供了模型评估、数据导入等额外的功能，还提供了一些更底层的机器学习原语。
- Spark GraphX：GraphX 是 Spark 面向图计算提供的框架与算法库。

## Spark Environment
### Spark 部署运行环境
**不同模式最主要的区别还是在指定--master时的运行环境和模式**
![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img0.png)
1. Local模式：本地环境：不需要其他任何节点资源就可以在本地执行 Spark 代码的环境
- 提交jar包到本地环境下运行的命令（官方案例中的计算pi）： 
  +   
    ```shell
        bin/spark-submit \
        --class org.apache.spark.examples.SparkPi \
        --master local[2] \
        ./examples/jars/spark-examples2.12-3.0.0.jar
    ```
  + 应用参数提交说明：
    ```text
        bin/spark-submit \ 
        --class \
        --master \ ... 
        # other options \ 
        [application-arguments] 
    ```
      1. --class	Spark 程序中包含主函数的类 
      2. --master	Spark 程序运行的模式(环境)	模式：local[*]、spark://linux1:7077、 Yarn
      3. --executor-memory 1G	指定每个 executor 可用内存为 1G 
      4. --total-executor-cores 2	指定所有executor使用的cpu核数。 为 2 个 
      5. --executor-cores	指定每个executor使用的cpu核数 
      6. application-jar	打包好的应用 jar，包含依赖。这 个 URL 在集群中全局可见。 比如 hdfs:// 共享存储系统，如果是file://path，那么所有的节点的 path 都包含同样的 jar 
      7. application-arguments 	传给 main()方法的参数 	
      
2. Standalone模式：经典的master-slave模式
   - 配置历史服务  history-server
   - ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img2.png)
   - 配置高可用HA（多个master）
     1. 停止集群 sbin/stop-all.sh 
     2. 启动 Zookeeper xstart zk 
     3. 修改 spark-env.sh 文件添加如下配置 
        * 注释如下内容：
        ```shell
           #SPARK_MASTER_HOST=linux1 
            #SPARK_MASTER_PORT=7077 
        ```
        * 添加如下内容: 
        ```shell
           SPARK_MASTER_WEBUI_PORT=8989 
           export SPARK_DAEMON_JAVA_OPTS=" 
           -Dspark.deploy.recoveryMode=ZOOKEEPER 
           -Dspark.deploy.zookeeper.url=hadoop102,hadoop103,hadoop104 
           -Dspark.deploy.zookeeper.dir=/spark" 
        ```

3. Yarn模式：
standalone模式有spark自身提提供计算资源，无需其他框架提供资源，这样降低了和其他第三方框架的耦合性，但是spark本身最主要用途是计算框架，而不是资源调度框架，所以还是用其他更专业的资源调度框架更靠谱。采用yarn进行资源调度。
    1. ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img3.png)
    2. ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img4.png)
    3. ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img5.png)
    4. ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img6.png)
    5. 提交应用的时候指定--master参数为yarn

4. k8s&mesos模式
容器化部署：容器管理工具中最为流行的就是 Kubernetes（k8s），而 Spark 也在最近的版本中支持了 k8s 部署模式。

5. windows模式：不用启用虚拟机直接使用



### Spark运行架构
**采用标准的主从结构master-slave**  
![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img7.png)
- Driver：用于执行Spark中的main方法 
- Executor：Spark Executor 是集群中运行在工作节点（Worker）中的一个 JVM 进程，是整个集群中 的专门用于计算的节点。资源一般指的是工作节点 Executor 的内存大小和使用的虚拟 CPU 核（Core）数量。
  + ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img8.png)
  + 并行和并发：并行为真的含有多个核同时运行，并发为一个核一直切换任务看起来像执行多个任务
  + Spark 为代表的第三代的计算引擎。第三代计算引擎的特点主要是 Job 内部的 DAG 支持（不跨越 Job），以及实时计算。 第一代为MR引擎要拆分算法，第二代为Tez等支持DAG的框架
  + ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img9.png)
- Matser&Worker
- ApplicationMaster
### 任务提交流程
- ![img.png](/blog_imgs/spark/base/spark deployment structure introduction/img10.png)
- Yarn Client模式：
  1. Client 模式将用于监控和调度的 Driver 模块在客户端执行，而不是在 Yarn 中，所以一般用于测试。     
  2. Driver 在任务提交的本地机器上运行    
  3. Driver 启动后会和 ResourceManager 通讯申请启动 ApplicationMaster
  4. ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，负 责向 ResourceManager 申请 Executor 内存    
  5. ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后 ApplicationMaster 在资源分配指定的 NodeManager 上启动 Executor 进程    
  6. Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行 main 函数    
  7. 之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个 stage 生 成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行。 

- Yarn Cluster模式 
  1.Cluster 模式将用于监控和调度的 Driver 模块启动在 Yarn 集群资源中执行。一般应用于 实际生产环境。
  2. 在 YARN Cluster 模式下，任务提交后会和 ResourceManager 通讯申请启动 ApplicationMaster
  3. 随后 ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster， 此时的 ApplicationMaster 就是 Driver。
  4. Driver 启动后向 ResourceManager 申请 Executor 内存，ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后在合适的 NodeManager 上启动 Executor 进程
  5. Executor进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行 main函数
  6. 之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个 stage 生 成对应的 TaskSet，之后将 task 分发到各个 Executor 上执行。

