# Hadoop

------

 ### 1. Hadoop 的组件

```sql
HDFS 
Yarn 
MapReduce
```



### 2. Hadoop 组成架构

![](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/20201108115105.png)

```sql
-- 1. NameNode(master) 管理者
		1.管理HDFS 的名称空间
		2.配置副本策略
		3.管理数据块的映射信息
		4.处理客户端的读写请求
-- 2. DataNode(slave) NameNode 下达指令,DataNode 执行实际的操作
		1. 存储实际的数据块
		2. 执行数据块的读写操作
-- 3. Client : 就是客户端
		1.文件切分 -> 文件上传到hdfs 的时候,client 将文件切分成一个个的block,然后上传
		2.与NanmeNode进行交互获取文件的位置信息
		3.与DataNode交互,读取或者写入数据
		4.提供命令管理HDFS,NameNode格式化
		5.提供命令访问HDFS,对HDFS 增删改查
-- 4. Secondy NameNode
		1.并不是NN的热备,并不能马上替换NameNode进行工作
		2.分担NN的工作量，定期和合并FsImage 和 Edits 文件,并推送给我们的NameNode
		3.紧急情况下,也可以恢复NameNode
		

```



### 3. 文件块大小

```sql
 --	 为什么是 128M？ 现在磁盘的读写速率是 100m/s 左右
     1.老版本 64M
     2.现在 128M ()
     
 --  为什么不能太大也不能太小？
 		 1. HDFS 块设置太小,会增加寻址时间, 一直找块开始的位置
 		 2. HDFS 块设置太大,磁盘传输数据的时间会大于 【定位这个块开始的时间】,会导致处理这块数据时非常缓慢
```



### 4.NameNode 的读写机制

**HDFS 写数据流程:**

![](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/20201108121515.png)

```sql
1. 客户端通过 distributed filesystem 模块向 NameNode 请求上传文件, NameNode 检查目录是否存在,父目录是否存在
2. NameNode 返回是否可以上传
3. 客户端请求第一个Block 上传到那个 dataNode, dn1,dn2,dn3
4. NameNode 返回三个DataNode 节点
5. 客户端通过 FS DataOutputStream 模块请求上传dn1 上传数据,dn1 收到请求 调用dn2,然后dn2调用 dn3,将通信管道建立
6. dn1,dn2,dn3 逐级应答客户单
7. 客户端开始往dn1 上传第一个block(先从磁盘读取放到一个本地内存缓存中)，以package 为单位，dn1收到后传递dn2,dn2 传递给dn3, 【每传一个package 会放在一个	应答队列中应答】
8. 当第一个block 传输完成之后,客户端请求NameNode 上传第二个block 服务器
```



**HDFS 读数据流程:**

![image-20201108123842718](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201108123842718.png)

```sql
1. 客户端通过 Distributed FileSystem 向 NameNode 请求下载文件. NameNode 通过查询元数据, 找到文件块所处的 DataNode 地址
2. 挑选一台DataNode(就近原则, 随机), 请求读取数据
3. DataNode 开始传输数据到客户端(从磁盘里面读取数据输入流, 以package为单位做数据效验)
4. 客户端以package 的方式来接受, 先写入本地缓存然后写入目标文件
```



### 5.网络拓扑-节点距离计算

```sql
-- NameNode 上传数据会选择距离比较近的 DataNode 来接受数据,最近的距离怎么来计算？

节点距离：两个节点到共同祖先距离总和
机架：子网接口

-- Hadoop 2.7.2 副本节点选择 ？
1. 第一个副本在client 所处节点上, 如果client 在集群外, 就随机选择一个
2. 第二个副本和第一个副本位于相同的机架, 随机节点
3. 第三个副本在不同机架上,随机节点

```



### 6.NameNode 和 SecondaryNameNode 

![image-20201108133102362](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201108133102362.png)

***

![image-20201108133501352](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201108133501352.png)

```sql
-- NN 和 2NN 工作机制
一. NameNode 中的元数据存储在哪里？
	 1. 磁盘 =>  随机访问     => 效率过低
   						响应客户的请求
   						
   2. 内存 => 断电,元数据丢失 => 集群无法工作
   3. 磁盘中备份元数据的 FsImage( 已经序列化的内容)
   4. 元数据更新, 同时也要更新FsImage, 导致效率过低，如果不更新，就会导致一致性问题
   5. NameNode断电，就会产生数据丢失
   6. 引入了Edits文件(只进行追加,效率很高), 元数据有更新, 修改内存中的元数据的时候并追加到Edits 中. 一旦断电, 通过FsImage 和 Edits 合并，合成元数据
   7. 长时间添加 Edits 中. 会导致文件数据过大，效率降低。断电恢复，恢复元数据时间过长, 需要定期合并FsImage 和 Edits 文件, 如果由NameNode 完成, 效率又				会比较低, 因此引入了SecondaryNameNode 专门用户FsImage和 Edits 的合并
   
二. checkpoint 时间设置
  1. SecondaryNameNode 每隔一个小时执行一次
  2. 一分钟检查一次操作次数, 当操作次数达到一百万,SecondaryNameNode 执行一次

```



### 7. DataNode 

#### 7.1 工作机制：

![image-20201108135949271](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201108135949271.png)

```sql
1. 一个数据块在DataNode 上以文件的形式存储在磁盘上, 包括两个文件,一个数据本身, 一个是元数据包括数据块的长度, 块数据的效验和以及时间戳
2. DataNode 启动后想NameNode 注册,通过后，周期性(一小时)向NameNode 上报所有块的信息
3. 心跳是每3秒一次,心跳返回的结果带有NameNode 改DataNode的命令:(如复制块数据到另外一台机器上, 删除某个数据块), 如果超过10分钟没有收到DataNode的心跳,则		认为该节点不可用
4. 集群运行中可以安全加入和退出一些机器


```

#### 7.2 数据完整性

![image-20201108140828141](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201108140828141.png)



```sql
-- DataNode 节点保证数据完整性的方法
1. DataNode 读取Block 的时候, 会计算CheckSum
2. 计算后的CheckSum, 与Block创建时值不一样, 说明Block 已经损坏
3. client 读取其他DataNode 上的Block
4. DataNode 在其文件创建后周期性验证 CheckSum
```

#### 7.3 掉线时限参数设置

![image-20201108141535584](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201108141535584.png)

```sql
需要注意的是hdfs-site.xml 配置文件中的heartbeat.recheck.interval的单位为毫秒，dfs.heartbeat.interval的单位为秒

<property>
    <name>dfs.namenode.heartbeat.recheck-interval</name>
    <value>300000</value>
</property>

<property>
    <name>dfs.heartbeat.interval</name>
    <value>3</value>
</property>
```

### 8. HDFS 2.X 新特性

```sql
-- 集群间数据拷贝
-- 小文件存档
		1. HDFS 小文件弊端
			a. 每个文件是按照块来存储的, 每个块的元数据存储在NameNode的内存中,因为大量小文件会耗尽NameNode 的大部分内存.
			b. 注意的是： 存储小文件需要的磁盘容量与数据块的大小无关。
		2. 使用HDFS 存档文件或HAR文件，文件存档工具，减少NameNode内存的同时,允许对文件进行透明的访问
    	a. HDFS存档文件对内还是一个个的文件,对NameNode 而言却是一个整体，减少了NameNode 的内存
-- 快照的管理
		1.   快照相当于对目录做一个备份。并不会立即复制所有文件，而是指向同一个文件。当写入发生时，才会产生新文件。
-- 回收站

```

### 9. HDFS-HA

![image-20201108155104459](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201108155104459.png)

```sql
-- Hadoop2.0 之前, HDFS 集群存在Name 存在单点故障(SPOF)
    1. NameNode 发生意外,集群无法使用,知道管理员重启
    2. NameNode 需要升级,软件、硬件升级, 此时集群无法使用
    
--  HDFS-HA 工作要点
		1. 元数据管理方式
			a. 内存中各存一份元数据
			b. Edits 日志只有Active状态的NameNode 可以写操作
			c. 两个NameNode 都可以读取Edits 
			d. 共享的Edits放在一个共享的存储中管理(qjournal 和 NFS 两种主流的实现)
		2. 需要一个状态管理的功能模块
			a. 需要实现一个zkfailover, 常驻在每一个NameNode节点上,每一个zkfailover负责监控自己的所有的NameNode节点,利用zk 做状态标识,当需要进行状态切换,由					zkfailover 来切换,切换时需要方式brain split 现象的发生
		3. 两个NameNode 之间能够ssh 无密码登录
		4. 隔离(Fence), 同一个时刻仅有一个NameNode 对外提供服务
-- HDFS 自动故障转移
		1. 增加两个组件： Zookeeper 和ZKFailoverController(ZKFC) 进程
		2. Zookeeper 维护少量的协调数据,通知客户端数据的改变和监视客户端故障的高可用
		3. HA 的故障转移依赖一下功能：
			a. 故障检测：NameNode 和 Zookeeper 维护一个持久会话. 机器崩溃,会话终止。Zookeeper 通知另外一个NameNode 故障转移
			b. 现役NameNode 选择：如果目前现役NameNode 崩溃, 另一个节点从Zookeeper获取排查锁来成为现役NameNode
		4. 每运行一个NameNode 主机也运行一个ZKFC进程,ZKFC 负责：
			a. 健康监测：ZKFC 使用健康检查命令定期的ping 相同主机的NameNode, 来标识节点的健康
			b. Zookeeper 会话管理: 当本地NameNode 健康，ZKFC 保持一个Zookeeper中打开的会话。本地的NameNode 处理Active 状态，ZKFC 也保持一个特殊的znode						锁,该锁使用Zookeeper 对短暂节点的支持，如果会话终止，锁节点将自动删除
			c.基于Zookeeper选择
			
-- Hadoop 2.X 
小文件归档:多个小文件一个地址空间
```

```sql
-- Hadoop 端口号：
nammnode: 50070
secondaryNameNode: 50090
datanode: 50010
fs.defaultfs:8020
yarn.resourcemanager.webapp.address:8088
历史服务器：198888
```

### 10. MapReduce 发生了多少次排序

```sql
总共发生四次排序：
1）Map 阶段：
环形缓冲区：对key 按照字段排序，排序手段是快排，
溢写到磁盘中，对多个溢写文件进行排序，排序手段是分区归并排序
2）reduce 阶段：
按照指定分区读取到reduce 缓存中(不够落盘)：归并排序
reduce Task 前分区排序：自定义
```

![image-20201118102020299](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118102020299.png)

![image-20201118102040844](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118102040844.png)

![image-20201118102130735](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118102130735.png)

```
Map Join:
a) Map Join适用于一张表十分小、一张表很大的场景
b) 在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。
提示：Map join 是MR的一种很好的优化手段，大家在复习Hadoop优化的时候可以将Hive优化联系起来，因为我们数仓中使用的依旧是MR引擎（其他提示：ORC）
```

| 压缩格式 | hadoop自带？ | 算法    | 文件扩展名 | 是否可切分 | 换成压缩格式后，原来的程序是否需要修改 |
| -------- | ------------ | ------- | ---------- | ---------- | -------------------------------------- |
| DEFLATE  | 是，直接使用 | DEFLATE | .deflate   | 否         | 和文本处理一样，不需要修改             |
| Gzip     | 是，直接使用 | DEFLATE | .gz        | 否         | 和文本处理一样，不需要修改             |
| bzip2    | 是，直接使用 | bzip2   | .bz2       | 是         | 和文本处理一样，不需要修改             |
| LZO      | 否，需要安装 | LZO     | .lzo       | 是         | 需要建索引，还需要指定输入格式         |
| Snappy   | 否，需要安装 | Snappy  | .snappy    | 否         | 和文本处理一样，不需要修改             |

提示：如果面试过程问起，我们一般回答压缩方式为Snappy，特点速度快，缺点无法切分

### 11 Yarn

1）Hadoop调度器重要分为三类：

FIFO 、Capacity Scheduler（容量调度器）和Fair Sceduler（公平调度器）。

Hadoop2.7.2默认的资源调度器是 容量调度器



容量调度器：支持多个队列源量进行限定，选择一个正在运行的任务数与其计算资源之间比值最小的队列(最闲的)，队列内任务按照作业优先级、提交时间、用户资源量限制和内存限制进行排序。

![image-20201118110823900](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118110823900.png)



公平调度器：支持多队列多用户，

1) 每个队列中的资源量可以配置，同一队列中的作业公平共享队列中所有资源，每个队列中的job按照优先级分配资源，优先级越高分配越多，

2) 在资源有限的情况下，每个job理想获得的计算资源与真实获得的计算资源的差值叫做缺额，同一队列中，job的资源缺额越大，越优先执行，可以多个任务同时运行。

资源不够所有的任务都运行不完



![image-20201118110941914](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118110941914.png)

### 13 Yarn 的 Job 提交流程

![image-20201118111412983](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118111412983.png)

### 14 Hadoop 优化

```sql
-- 小文件的处理
合并小文件：
(1)对小文件归档成(har)， 自定义inputformat 将小文件存储成sequenceFile 
(2) 输入的时候: CombinerFIleInputFormat 做为输入端小文件的问题
(3）大量的小文件可以开启JVM 重用

map阶段
（1）增大环形缓冲区大小。由100m扩大到200m
（2）增大环形缓冲区溢写的比例。由80%扩大到90%
（3）减少对溢写文件的merge次数。
（4）不影响实际业务的前提下，采用combiner提前合并，减少 I/O。
 
reduce阶段
（1）合理设置map和reduce数：两个都不能设置太少，也不能设置太多。太少，会导致task等待，延长处理时间；太多，会导致 map、reduce任务间竞争资源，造成处理超时等错误。
（2）设置map、reduce共存：调整slowstart.completedmaps参数，使map运行到一定程度后，reduce也开始运行，减少reduce的等待时间。
（3）规避使用reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。
（4）增加每个reduce去map中拿数据的并行数
（5）集群性能可以的前提下，增大reduce端存储数据内存的大小。
 
IO传输
（1）采用数据压缩的方式，减少网络IO的的时间。安装Snappy和LZOP压缩编码器。
（2）使用SequenceFile二进制文件

整体
（1）MapTask默认内存大小为1G，可以增加MapTask内存大小为4-5g
（2）ReduceTask默认内存大小为1G，可以增加ReduceTask内存大小为4-5g
（3）可以增加MapTask的cpu核数，增加ReduceTask的cpu核数
（4）增加每个container的cpu核数和内存大小
（5）调整每个Map Task和Reduce Task最大重试次数


```

