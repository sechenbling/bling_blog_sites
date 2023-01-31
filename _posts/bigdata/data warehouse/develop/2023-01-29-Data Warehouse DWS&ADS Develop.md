---
title: Data Warehouse DWS&ADS Develop
author: Lynn
date: 2023-01-27 13:30:00 +0800
last_modified_at: 2023-01-27 13:35:00 +0800
categories: [Bigdata, DataWarehouse]
tags: [data warehouse]
syntax: colorful
---
## DWS层
汇总数据层，由明细数据汇总而来的数据，基于上层的指标需求，构建公用的中间的统计结果，为了减少重复计算。

### 设计过程
- DWS层的设计参考指标体系。
  + 原子指标，派生指标，衍生指标
- DWS层的数据存储格式为orc列式存储+snappy压缩。
- DWS层表名的命名规范为dws_数据域_统计粒度_业务过程_统计周期（1d/nd/td）注：1d表示最近1日，nd表示最近n日，td表示历史至今。
  + 统计粒度：通俗点就是表里的用来分组的字段（字段数并不固定）
  
## ADS层
根据具体业务需求来获取对应的数据，存放各项统计指标标准

### 开窗函数：
区分开窗函数和聚合函数 
- 聚合函数：按照字段进行分组(group by)进行shuffle，将相同码的数据进行聚合，然后得到每一组的KV键值（多行聚合一行） 
- 开窗函数：和聚合函数很类似，按照partition by 的字段进行shuffle，将相同key的数据都缓存起来（多行“聚合”多行），前一个多行是可以人为指定大小的，后一个多行值得是对筛选数据进行窗口后的多个结果 
- 补充：这些窗口的划分都是在分区内部！超过分区大小就无效了，可以看到如果不指定ROWS BETWEEN,默认统计窗口为从起点到当前行;关键是理解 ROWS BETWEEN 含义,也叫做window子句：
    1. PRECEDING：往前
    2. FOLLOWING：往后 
    3. CURRENT ROW：当前行 
    4. UNBOUNDED：无边界
    5. UNBOUNDED PRECEDING 表示从最前面的起点开始
    6. UNBOUNDED FOLLOWING：表示到最后面的终点

```hql
CREATE TABLE student_scores(
    id int,
    studentId int,
    language int,
    math int,
    english int,
    classId string,
    departmentId string
);

INSERT INTO table student_scores VALUES
  (1,111,68,69,90,'class1','department1'),
  (2,112,73,80,96,'class1','department1'),
  (3,113,90,74,75,'class1','department1'),
  (4,114,89,94,93,'class1','department1'),
  (5,115,99,93,89,'class1','department1'),
  (6,121,96,74,79,'class2','department1'),
  (7,122,89,86,85,'class2','department1'),
  (8,123,70,78,61,'class2','department1'),
  (9,124,76,70,76,'class2','department1'),
  (10,211,89,93,60,'class1','department2'),
  (11,212,76,83,75,'class1','department2'),
  (12,213,71,94,90,'class1','department2'),
  (13,214,94,94,66,'class1','department2'),
  (14,215,84,82,73,'class1','department2'),
  (15,216,85,74,93,'class1','department2'),
  (16,221,77,99,61,'class2','department2'),
  (17,222,80,78,96,'class2','department2'),
  (18,223,79,74,96,'class2','department2'),
  (19,224,75,80,78,'class2','department2'),
  (20,225,82,85,63,'class2','department2');

SELECT studentId,math,departmentId,classId,
  count(math) over() as count1,
  count(math) over(partition by classId) as count2,
  count(math) over(partition by classId order by math) as count3,
  count(math) over(partition by classId order by math rows between 1 preceding and 2 following) as count4
FROM student_scores 
WHERE departmentId='department1';
```
{: file='window_example.sql'}

```text
studentid   math    departmentid    classid count1  count2  count3  count4
111         69      department1     class1  9       5       1       3
113         74      department1     class1  9       5       2       4
112         80      department1     class1  9       5       3       4
115         93      department1     class1  9       5       4       3
114         94      department1     class1  9       5       5       2
124         70      department1     class2  9       4       1       3
121         74      department1     class2  9       4       2       4
123         78      department1     class2  9       4       3       3
122         86      department1     class2  9       4       4       2
```
{: file='result.csv'}
		
		
		
		
	
	
