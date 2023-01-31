---
title: Data Warehouse Design
author: Lynn
date: 2023-01-28 22:00:00 +0800
last_modified_at: 2023-01-28 22:00:00 +0800
categories: [Bigdata, DataWarehouse]
tags: [data warehouse]
syntax: colorful
---

## 数据仓库设计

### 数据仓库分层规划
使数据体系更加清洗，简单化复杂问题（one data理论分层）
![img.png](/blog_imgs/bigdata/data warehouse/theory/data warehouse design/data warehouse design img1.png)
ODS:
: operation data store:业务系统的原始数据，未经处理的数据，放在HDFS以文件形式的存储，从HDFS原路径load(底层是剪切操作)到ODS层。

DWD:
: data warehouse detail:基于维度建模理论进行构建，保存各业务过程中最小粒度的操作记录，存放事实表。

DIM:
: dimension:基于维度建模理论进行构建，存放维度表，保存一致性维度信息。

DWS:
: data warehouse summary:需求驱动来建立对应的表，以分析的主题对象作为建模驱动构建公共统计粒度（不同需求之间通用的计算结果，减少重复计算）的汇总表。

ADS:
: application data service:存放各项统计指标结果。

## 数仓构建流程
![img.png](/blog_imgs/bigdata/data warehouse/theory/data warehouse design/data warehouse design img2.png)

### 数据调研：
- 业务调研：熟悉业务流程，熟悉业务数据
- 需求分析：明确需求所需的业务过程及维度

### 明确数据域
按照特定标准对数据进行纵向切分，划分数据域便于数据的管理和应用
- 比如电商业务过程中将业务分为交易域，流量域，用户域，互动域等多个域
- 说白其实对应DWD层的事实表，DWS的汇总表来自事实表，其实也来自数据域

|  数据域 |         业务过程                            |
|:------:|:-----------------------------------------:|
| 交易域  | 加购、下单、取消订单、支付成功、退单、退款成功     |
| 流量域  | 页面浏览、启动应用、动作、曝光、错误             |
| 用户域  | 注册、登录                                  |
| 互动域  | 收藏、评价                                  |
| 工具域  | 优惠券领取、优惠券使用（下单）、优惠券使用（支付）  |

### 构建业务总线矩阵
服务于维度模型的设计
- ![img.png](/blog_imgs/bigdata/data warehouse/theory/data warehouse design/data warehouse design img3.png)
- 总线矩阵中通常只包含事务型事实表，其他两种类型的表单独设计

### 维度模型设计
参照业务总线矩阵，构建事实表存储在DWD层，维度表存储在DIM层

### 明确统一指标
整理出指标体系理论，包括原子指标、派生指标、衍生指标，主要是为了指标定义标准化，遵循一套标准，避免重复定义和定义歧义问题
- 原子指标：基于某一业务过程的度量值，原子指标核心功能就是对指标的聚合逻辑进行定义
  + 比如每个订单都有金额，订单总额就是一个原子指标
  + 原子指标只是用来辅助下面两个定义的一个概念，通常没有实际的对应需求
- 派生指标：基于原子指标，通常会对应实际的统计需求，关系图如下
- ![img.png](/blog_imgs/bigdata/data warehouse/theory/data warehouse design/data warehouse design img4.png)
- 衍生指标：在多个派生指标的基础上，通过各种逻辑运算复合而来的，比如比率、比例等
- ![img.png](/blog_imgs/bigdata/data warehouse/theory/data warehouse design/data warehouse design img5.png)
- 指标理论体系意义：公共的派生指标保存在DWS层，统计需求足够多的时候，会出现公共的派生指标，因此DWS层设计可以参考现有的统计需求整理出来 

### 汇总模型设计
一张汇总表会包括业务过程相同，统计周期相同，统计粒度相同的多个派生指标
- ![img.png](/blog_imgs/bigdata/data warehouse/theory/data warehouse design/data warehouse design img6.png)
- 汇总表是由一张事实表（一个业务过程）汇总而来的，汇总表却不止一张，属于一对多的关系

## 数据仓库环境配置
绝大多数与事实表的业务表都是增量同步，绝大多数与维度表相关的业务表都是全量同步（非绝对，比如需要设计成拉链表类型的维度表不需要同步所有数据，只需要同步新增变化，用增量同步即可）

### **数仓运行环境准备**
- Hive环境：引擎变更，从MR转变为使用Spark
  + 区分
    * Hive on Spark（语法是Hive语法，执行引擎变为Spark）；Spark采用RDD执行。除了数仓的建模和SQL以外对数据安全和认证以及元数据管理的周边数仓任务框架支持更好，生态更好。
      * Spark on Hive（语法是Spark，Spark负责SQL解析，Hive作为存储元数据）；Spark采用RDD执行。效率更高但是生态较差。
      * 两者都可以作为数仓运行环境；
    * 记得要重新编译Hive的源码，使其内部能够使用Spark3.0.0，在Hive部署的节点上部署Spark
    * Hive on spark是以一个会话作为一个资源分配，一个会话为每次开启的Hive 相较于使用MR每个mr程序一个资源分配，开启会话的时候都比较忙，但是后续操作spark引擎会快于mr引擎
- Yarn环境：增加Application Master资源比例，虚拟机集群环境资源较少，默认只分配10%资源给一个Job会导致同一时刻只能运行一个Job的情况（资源总量太少，10%就更少了，内存资源可能只够一个任务运行），适当调大AM可以使用的资源比例使其使用更多资源
  ```xml
  <property>
    <name>yarn.scheduler.capacity.maximum-am-resource-percent</name>
    <value>0.1</value> <!-- 修改为80% -->
    <description>
        Maximum percent of resources in the cluster which can be used to run application masters i.e. controls number of concurrent running applications.
    </description>
  </property>
  ```
  {: file='$HADOOP_HOME/etc/hadoop/yarn-site.xml'}

### **数仓开发环境准备**
- 使用DBeaver或者DataGrip，两者都需要使用JDBC协议连接到Hive，所以需要启动HiveServer2服务(Hiveserver2就是用于提供JDBC接口)，使用DataGrip连接到Hive即可。
			
