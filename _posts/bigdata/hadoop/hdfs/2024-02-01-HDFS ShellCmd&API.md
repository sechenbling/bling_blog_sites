---
title: HDFS ShellCmd&API
author: Lynn
date: 2024-02-01 22:35:00 +0800
last_modified_at: 2024-02-01 22:35:00 +0800
categories: [Bigdata, Hadoop]
tags: [hdfs]
syntax: colorful
---

## HDFS 命令

`hadoop fs -xxx`和`hdfs dfs -xxx` 最终效果和命令都是一样的 两个都可以使用

```text
hadoop fs -moveFromLocal /local /remote	从本地剪切粘贴到 HDFS
hadoop fs -copyFromLocal /local /remote 	从本地文件系统中拷贝文件到 HDFS 路径去
hadoop fs -put /local /remote	-put：等同于 -copyFromLocal
hadoop fs -appendToFile /local /remote	追加一个文件到已经存在的文件末尾
hadoop fs -copyToLocal /remote	从 HDFS 拷贝到本地
hadoop fs -get /remote	等同于 copyToLocal
hadoop fs -ls /remote	显示目录信息
hadoop fs -cat /remote	显示文件内容
hadoop fs -mkdir /remote	创建路径
hadoop fs -cp /remote1 /remote2	从 HDFS 的一个路径拷贝到 HDFS 的另一个路径
hadoop fs -mv /remote1 /remote2	在 HDFS 目录中移动文件
hadoop fs -tail /remote	显示一个文件的末尾 1kb 的数据
hadoop fs -rm /remote	删除文件
hadoop fs -rm -r /remote	递归删除目录及目录里面内容（文件夹
hadoop fs -du	统计文件夹下每个目录或文件的大小信息
hadoop fs -du -s	统计该文件夹整体大小
hadoop fs -du -h	统计文件夹下每个目录或文件的大小信息 (存在单位换算 B to KB MB GB…)
hadoop fs -du -s -h	统计该文件夹整体大小(存在大小换算
hadoop fs -chgrp	hadoop fs -chgrp group 修改组
hadoop fs -chmod	hadoop fs -chmod 777 修改三类用户权限
hadoop fs -chown	hadoop fs -chown chen:chen 修改用户
```

## HDFS 命令
```java
@Before
public void init() throws IOException, URISyntaxException, InterruptedException{
    //连接集群NameNode的地址
    URIuri=newURI("hdfs://hadoop102:8020");

    //创建一个配置文件
    Configurationconfiguration=newConfiguration();

    //获取用户信息
    Stringuser="chen";

    //获取客户端对象
    fs=FileSystem.get(uri,configuration,user);

    //创建一个文件夹操作
    fs.mkdirs(newPath("/xiyou/huaguoshan"));
}

@After
public void close() throws IOException{
    //关闭资源
    fs.close();
}
```

配置文件中的dfs.replication参数为上传后的副本数，这个参数的生效优先级为代码中的configuration.set("dfs.replication",xx) > 项目resource目录下的hdfs-site.xml文件（文件内部增加dfs.replication参数）  >  服务器上的hdfs-site.xml(修改文件内的dfs.replication参数)  > 服务器上的hdfs-default.xml(默认无法修改)

