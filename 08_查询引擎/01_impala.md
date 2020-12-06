# Impala

***



### 1. Impala 概念

```sql
-- 1.什么是Impala 
Cloudera公司推出，提供对HDFS、Hbase数据的高性能、低延迟的交互式SQL查询功能。
基于Hive，使用内存计算，兼顾数据仓库、具有实时、批处理、多并发等优点。
是CDH平台首选的PB级大数据实时查询分析引擎。
```

### 2. Impala 优缺点

> ![image-20201109211124733](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109211124733.png)



### 3. Impala 的组成

> ![image-20201109211223533](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109211223533.png)



### 4. Impala 运行原理

> ![image-20201109211319422](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109211319422.png)

```sql
Impala执行查询的具体过程：
1）当用户提交查询前，Impala先创建一个负责协调客户端提交的查询的Impalad进程，该进程会向Impala State Store提交注册订阅信息，State Store会创建一个statestored进程，statestored进程通过创建多个线程来处理Impalad的注册订阅信息。
2）用户通过CLI客户端提交一个查询到impalad进程，Impalad的Query Planner对SQL语句进行解析，生成解析树；然后，Planner把这个查询的解析树变成若干PlanFragment，发送到Query Coordinator.
3）Coordinator通过从元数据库中获取元数据，从HDFS的名称节点中获取数据地址，以得到存储这个查询相关数据的所有数据节点。
4）Coordinator初始化相应impalad上的任务执行，即把查询任务分配给所有存储这个查询相关数据的数据节点。
5）Query Executor通过流式交换中间输出，并由Query Coordinator汇聚来自各个impalad的结果。
Coordinator把汇总后的结果返回给CLI客户端。

```

### 5.Impala 数据类型

| Hive数据类型 | Impala数据类型 | 长度                                                 |
| ------------ | -------------- | ---------------------------------------------------- |
| TINYINT      | TINYINT        | 1byte有符号整数                                      |
| SMALINT      | SMALINT        | 2byte有符号整数                                      |
| INT          | INT            | 4byte有符号整数                                      |
| BIGINT       | BIGINT         | 8byte有符号整数                                      |
| BOOLEAN      | BOOLEAN        | 布尔类型，true或者false                              |
| FLOAT        | FLOAT          | 单精度浮点数                                         |
| DOUBLE       | DOUBLE         | 双精度浮点数                                         |
| STRING       | STRING         | 字符系列。可以指定字符集。可以使用单引号或者双引号。 |
| TIMESTAMP    | TIMESTAMP      | 时间类型                                             |
| BINARY       | 不支持         | 字节数组                                             |

```sql
注意：Impala虽然支持array，map，struct复杂数据类型，但是支持并不完全，一般处理方法，将复杂类型转化为基本类型，通过hive创建表
```

### 6. 优化

```sql
1、尽量将StateStore和Catalog单独部署到同一个节点，保证他们正常通行。
2、通过对Impala Daemon内存限制（默认256M）及StateStore工作线程数，来提高Impala的执行效率。
3、SQL优化，使用之前调用执行计划
4、选择合适的文件格式进行存储，提高查询效率。
5、避免产生很多小文件（如果有其他程序产生的小文件，可以使用中间表，将小文件数据存放到中间表。然后通过insert…select…方式中间表的数据插入到最终表中）
6、使用合适的分区技术，根据分区粒度测算
7、使用compute stats进行表信息搜集，当一个内容表或分区明显变化，重新计算统计相关数据表或分区。因为行和不同值的数量差异可能导致impala选择不同的连接顺序时，表中使用的查询。
[hadoop104:21000] > compute stats student;
Query: compute stats student
+-----------------------------------------+
| summary                                 |
+-----------------------------------------+
| Updated 1 partition(s) and 2 column(s). |
+-----------------------------------------+
8、网络io的优化：
      –a.避免把整个数据发送到客户端
      –b.尽可能的做条件过滤
      –c.使用limit字句
–d.输出文件时，避免使用美化输出
–e.尽量少用全量元数据的刷新
9、使用profile输出底层信息计划，在做相应环境优化
```

