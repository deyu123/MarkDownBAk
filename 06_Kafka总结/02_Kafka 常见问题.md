# Kafka 常见问题

### 1. 数据量

```sql
每天 8 T , 9 万条/s, 高峰 1000万，日志在1k 左右, 每秒 1G

基于发布和订阅的消息队列
```

### 2. 常见问题

```sql
-- kafka 消费能力不足
1.增加topic 分区数，提高消费者组消费的数量，消费者组数=分区数
2.下游处理数据不及时，提高每批次拉取的数量

-- kafka 挂掉，日志服务器有记录可以重跑

-- 消费方式
consumer => pull => 拉去数据(适当的速率消费信息)
	假如使用 => push => 1.很难使用不同速率的消费者
										 2. 且以最快的速度传递信息
										 consumer 来不及处理的话 => 拒绝服务,网络拥堵
	pull 也有不足 => 1.消费者可能陷入循环一直返回空数据
									2.消费数据的时候,会传入时间参数 timeout， 如果没有可供消费的，cousume 会等待一段时间再返回，时间长度就是timeout 
									
-- 分区分区策略
概念：topic => partition => 由哪个cousumer 来消费
两种策略：   1. range 
				   2.  roundrobin
				   
-- offset 的维护
0.9 之前在zk 中
0.9 之后内置在topic 中，topic 名： _consumer_offsets

-- 什么时候会触发rebalance 
1. 已有的消费者退出消费者组
2. 订阅的主题发生变化
3. 触发分区重新分配

-- 消费者分区发生变化
1. 获取自己被重新分配的分区
2. 定位最低提交的offset位置重新消费

-- 拦截器： 定义化需求，修改消息，多个拦截器对同一个消息形成拦截器链

-- kafka controlle 的作用
1. 集群的上下线
2. topic 分区副本的分配，topic的创建
3. leader 的选举

-- HMaster 的作用
1. 表操作：create,delete,alter
2. regionServer: 分配region 到regionServer
								 监控regionServer 状态
								 负载均衡，故障转移


-- kafka controller 的选举机制
1. 利用zk 的临时节点来实现分布式锁，谁先抢，谁做controlle, Hbase HMaster , canal 高可用都是使用分布式锁
```

### 3.基础架构

```
1.topic -> partition (方便集群的扩展) 
2.分区 -> 设计 -> 消费者组的概念(并行消费)
3.可用性 -> partition -> 副本->NA 
										 -> .log 文件 produce 生产的文件追加到文件的末端
										 -> .index 索引文件
					 每条数据都有自己的offset , 出错后从上次的位置开始消费
					 
produce  -> partition -> ISR  => 
																1. (leader 收到数据，follewer 开始同步， 有一个follewer开始故障，leader 开始等)					 
																2. replica.time.max.ms 超过这个时间阈值 选择新的leader 	
ack =>  0 : 不等直接返回
				1 : leader 同步完成后返回
				-1: leader 和follewer 都同步后开始返回
				
幂等性： enable.idemptance = true , 自动开启ack =-1 ,		retries -> Integer.max.value		
```

### 4.为什么这么快

```sql
-- 顺序写磁盘
顺序写 600m/s, 磁盘的机械结构，顺序写会省去大量磁盘寻址的时间

-- 0拷贝技术

-- batch.size linger.ms
达到一定的batch 
达到一定的time 开始发送
```



### 5. 常见问题

```sql
-- 日志保留时间 
7天

-- kafka 分区数
分区数并不是越多越好，一般分区不好超过集群机器数量，分区数越大，占用的内存越大
一个节点集中的分区越多，当他宕机时，对系统的影响就越大
分区数一般设置 3-10 个

副本数： 2个
topic 多少个日志类型，多少个topic 


Ack=0，相当于异步发送，消息发送完毕即offset增加，继续生产。
Ack=1，leader收到leader replica 对一个消息的接受ack才增加offset，然后继续生产。             
Ack=-1，leader收到所有replica 对一个消息的接受ack才增加offset，然后继续生产

Kafka的ISR副本同步队列:
ISR（In-Sync Replicas），副本同步队列。ISR中包括leader和follower。如果Leader进程挂掉，会在ISR队列中选择一个服务作为新的Leader。有replica.lag.max.messages和replica.lag.time.max.ms两个参数决定一台服务是否可以加入ISR副本队列，在0.10版本移除了replica.lag.max.messages参数，防止服务频繁的进去队列

Kafka 分区分配策略：

在 Kafka内部存在两种默认的分区分配策略：Range和 RoundRobin。
Range 是默认策略。Range是对每个Topic而言的（即一个Topic一个Topic分），首先对同一个Topic里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。然后用Partitions分区的个数除以消费者线程的总数来决定每个消费者线程消费几个分区。如果除不尽，那么前面几个消费者线程将会多消费一个分区。
例如：我们有10个分区，两个消费者（C1，C2），3个消费者线程，10 / 3 = 3而且除不尽。
C1-0 将消费 0, 1, 2, 3 分区
C2-0 将消费 4, 5, 6 分区
C2-1 将消费 7, 8, 9 分区
RoundRobin：
前提：同一个Consumer Group里面的所有消费者的num.streams（消费者消费线程数）必须相等；每个消费者订阅的主题必须相同。
第一步：将所有主题分区组成TopicAndPartition列表，然后对TopicAndPartition列表按照hashCode进行排序，最后按照轮询的方式发给每一个消费线程。

kafka 中的数据量：
10G数据 = 10万日活 * 100条 *1k=  1000万条*1k

每天总数据量10g，每天产生1000万条日志，每秒钟1000万/24/60/60=115条
每平均秒钟：115条
低谷每秒钟：40条
高峰每秒钟：115条*（2-20倍）=230条-2300条
每条日志大小：0.5k-2k
每秒多少数据量：230k-2MB
```



