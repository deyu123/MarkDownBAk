# Kafka总结

***************

## 一、kafka概述

### 1.1 kafka定义

> Kafka是一个分布式的基于**发布/订阅**模式的**消息队列，**主要应用于大数据**实时**处理领域。
>
> 订阅式模式：一对多的关系，一个生产者，数据存储在消息队列中，多个消费者均可从这个消息对列中获取数据，**消费者消费数据之后不会清除消息。**

### 1.2 框架说明

> 一般都是从命令行和API两个方面进行讲解。
>
> 数据处理框架需要从数据的安全性以及效率两个方面深入了解。

### 1.3 Kafka涉及的关键词

```sql
1. producer: 消息的生产者，即为向kafka broker发消息;
2. broker ： kafka集群的节点；
3. topic : 队列（话题），生产者和消费者面向的都是一个topic；
4. message：消息，队列中的一条消息；
5. partition: 分区，为方便扩展和提高吞吐量，将一个topic分为了多个partition；
6. index ： 消息数据在log文件中的索引；
7. log ：消息的具体数据；
8. timeindex： 时间索引，代表发送的数据时间索引；
9. offset ： 消息的偏移量，每一条消息都对应一个offset；
10. segment : 一个分片数据；
11. leader ：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader；
12. follower : 每个分区多个副本中的“从”，实时从leader中同步数据，保持和leader数据的同步。leader发生故障时，某个follower会成为新的leader
```

## 二、Kafka安装

### 2.1 集群部署

#### 2.2.1  解压安装包

在/opt/software目录下

```sq1
tar -zxvf kafka_2.11-2.4.1.tgz -C /opt/module/
```

#### 2.2.2  修改解压后的文件名称

```sql
 mv kafka_2.11-2.4.1/ kafka
```

#### 2.2.3 创建logs文件夹

在/opt/module/kafka目录下

```sql
mkdir logs
```

#### 2.2.4 修改配置文件

/opt/module/kafk路径下 

```sql
vim config/server.properties
```

修改如下三个参数，修改后的值如下：

```sql
broker.id=2；
log.dirs=/opt/module/kafka/logs
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka
```

#### 2.2.5 配置环境变量

```Sql
sudo vim /etc/profile.d/my_env.sh
```

增加如下配置：

```sql
#KAFKA_HOME
KAFKA_HOME=/opt/module/kafka
PATH=$PATH:$KAFKA_HOME/bin
export KAFKA_HOME
```

生效配置文件：

```sql
source /etc/profile
```

#### 2.2.6  分发安装包

```sql
xsync kafka/
```

注意：分发之后记得配置其他机器的环境变量

#### 2.2.7 修改其他机器的配置文件

- /opt/module/kafka/config/server.properties中的broker.id=3、broker.id=4

  注：broker.id不得重复

#### 2.2.8  启动集群

1. 首先启动zookeeper集群和hadoop集群

2. 依次在hadoop102、hadoop103、hadoop104节点上启动kafka

```sql
kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties
```

- -daemon属于后台启动，没有-daemon则为前台启动

#### 2.2.9 关闭集群

```Sql、
kafka-server-stop.sh
```

#### 2.2.10 kafka群起群停脚本

```sql
#！bin/bash
if [ $# -lt 1 ]
 then
   echo "No Args Input Error"
   exit
fi
case $1 in
"start")
for i in `cat /opt/module/hadoop-3.1.3/etc/hadoop/workers`
do
echo "==========start $i kafka=========="
ssh $i '$KAFKA_HOME/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties'
done
;;
"stop")
for i in `cat /opt/module/hadoop-3.1.3/etc/hadoop/workers`
do
echo "==========stop $i kafka=========="
ssh $i '$KAFKA_HOME/bin/kafka-server-stop.sh'
done
;;
*)
  echo "Input Args Error"
;;
esac
```

### 2.2 Kafka命令操作

#### 2.2.1 查看当前服务器中的所有topic

```sql
kafka-topics.sh  --bootstrap-server hadoop102:9092 --list
```

选项说明：

- --list ：查看kafka所有的topic
- --bootstrap-server : 连接kafka集群
- --hadoop102:9092：hadoop102是指连接kafka任意一台机器，9092：kafka内部通信的端口

#### 2.2.2  创建topic

```sql
kafka-topics.sh  --bootstrap-server hadoop102:9092 --create --topic first --partitions 2 --replication-factor 2
```

选项说明：

- --topic  : 定义topic名字
- --partitions  : 定义分区数 
- --replication-factor : 定义副本数

#### 2.2.3 查看某个Topic的详情

```sql
 kafka-topics.sh --bootstrap-server hadoop102:9092 --describe --topic first
```

选项说明：

- --topic first ： 查看指定的话题，如果不加此选项，则表示查看所有的话题

#### 2.2.4  修改分区数

```sql
kafka-topics.sh --bootstrap-server hadoop102:9092 --alter --topic first --partitions 6
```

说明：

- --分区数只能增加不能减少
- 分区内部消息有序，分区之间消息无序

#### 2.2.5 发送消息

```sql
kafka-console-producer.sh --broker-list  hadoop102:9092,hadoop103:9092,hadoop104:9092 --topic first


>hello world
>atguigu  atguigu
```

选项说明：

- hadoop102:9092,hadoop103:9092,hadoop104:9092 : kafka的集群中的broker，其实写一个也是可以的，写3个的目的只是避免当连接的kafka集群broker故障时连不上kafka集群的情况。

#### 2.2.6  消费消息

```sql
kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first
```

选项说明：

- --from-beginning ：

  加上：会把topic中以往所有的数据都读取出来

  不加：此时只会消费最新的数据，原来topic中的数据不会被消费

#### 2.2.7  删除topic

```sql
kafka-topics.sh --bootstrap-server hadoop102:9820 --delete --topic first
```

## 三、 Kafka深入流程

说明：此框架步步引导，采取提出问题解决问题的方式阐述。

### 3.1 Kafka工作流程及文件存储机制

![图2](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200529005434.png)

1.  kafka 以topic（话题）为单位，每一个话题分为多个区（创建话题的时候指定分区的个数），每个分区中存储的数据是不一样的，同时每个分区的数据在其他分区也会创建副本。
2.  不同的分区分布在kafka集群不同的机器（broker，代理人）上面；
3.  消息的生产和消费均是分区为单位；
4.  分区内的数据是有序的，分区之间的顺序是无序的；
5.  offset 指消息的偏移量；
6.  每个分区都是一个文件夹，文件中包含index（数据在log中的索引）、log（真实的数据）、timeindex (数据发送的时间索引) ，时间索引和index索引均是用来提高查询数据效率；
7.  当产生新的数据以后会向log文件中进行追加，同时index和timeindex也会增加；
8.  Kafka采取了**分片**和**索引**机制。
9.  topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，且每条数据都有自己的offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个offset，以便出错恢复时，从上次的位置继续消费

![图1](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200610151457.png)

```sql
-- 灵魂拷问1：
产生的数据一直向同一个log中进行追加，会有什么问题呢？

-- 答案：
log中的数据会越来越大，查询和读取效率会变慢。

-- 解决方式：
数据达到一定程度以后（默认值为1G：log.segment.bytes = 1G），log会进行数据切分，生成多个segment切分文件。
切分后的文件依然包含index、log、timeindex 。所以三个文件是作为一个整体的。 --切分机制

```

![image-20200714205920775](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200714205928.png)

![image-20200714210619003](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200714210619.png)



> 切分的文件位于同一个文件夹下，该文件夹的命名规则为：==topic名称+分区序号==。
>
> 例如，first这个topic有三个分区，则其对应的文件夹为first-0,first-1,first-2

```sql
-- 灵魂拷问2：
现在假如有两个切分的文件，当有一个消费者需要消费一条消息（假如是 offset = 3），怎么知道这个消息在哪个切分文件中，以及真实数据如何查询？

--答案：
1）log和index文件名说明： -- 牢记log、index、timeindex是一个整体
index：00000000000000000000.index
log：00000000000000000000.log
前面的数字00000000000000000000：代表此log文件中第一条消息的offset。
'.index文件存储大量的索引信息，“.log”文件存储大量的数据，索引文件中的元数据指向对应数据文件中message的物理偏移地址'
2） 查询的方式：
根据消费消息的offset值 -->找到指定的index文件 --> 匹配此条消息在log文件中数据的偏移量（即该数据在log文件中起始位置）--> 找到待消费的数据 

```

```sql
-- 灵魂拷问3：
为什么kafka要采取向一个log文件中追加数据呢？

-- 答案：
1）减少IO；
2）消费数据是连续进行消费，连续读取数据的效率高。
```

### 3.2 Kafka之生产者producer

#### 3.2.1 分区策略

1. 首先producer发送的数据会被封装成 ProducerRecord 对象，根据对象的参数，分区情况如下：
   - 情况1 ：指定了partition；

   - 情况2 ：未指定partition，封装key，则按照key的hashcode % 分区数量 得出在哪个分区；

   - 情况3：未指定partition，也未封装key处理方式 :

     参数1：producer发送的数据量：batch.size，默认值为16Kb；

     条件2：linger.ms：两条数据发送的间隔时间 t ，默认值为0s；

     当发送的数据量 < batch.size 并且 发送的数据时间间隔  < t   时，所有的数据在一个分区；

     当发送的数据量 > batch.size 或者 发送的数据时间间隔  >  t 时，则数据会进入下一个分区；

     分区与分区之间采取轮询的方式。

     ![图4](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200529005453.png)

     ![图3](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200529005450.png)

#### 3.2.2 数据可靠性保证

数据传输流程：

producer -----> server（kafka） --------->消费者

- 过程1：producer -----> server（kafka）

```sql
-- 灵魂拷问1： 
如何保证从producer发送数据server的过程中数据不丢失？

-- 答案：
server收到数据以后会回执，发送ack（acknowledgement确认收到）给producer，producer收到ack以后，则确定数据传送的过程中没有丢失。
```

- 过程2 ： server的数据存储过程

```sql
-- 灵魂拷问2：
如何确保数据在server中能够被妥善保管呢？

-- 答案：
server向producer回执ack的时机：
模式1：leader收到消息以后立即回复ack；
模式2：leader收到消息并存储在本地以后，立即回复ack；
模式3：leader收到消息后，所有follow从leader中拉取数据，当所有的follower完成存储以后，leader向producer回复ack。

说明：情况1/2/3是通过acks参数进行配置。
acks=0 -- leader收到消息以后立即回复ack
acks=1 -- leader收到消息并存储在本地以后，立即回复ack
acks=-1或all --leader收到消息后，所有follow从leader中拉取数据，当所有的follower完成存储以后，leader向producer回复ack
-- 默认情况下是acks=1；

===============================================================================
-- 灵魂拷问3：
leader与follower副本数据同步策略是什么呢？

-- 答案
两种副本同步策略。
第一种：半数以上完成同步，就发送ack
第二种：全部完成同步，才发送ack

'kafka选择全部完成同步，才发送ack'

================================================================================
-- 灵魂拷问4：
kafka选择第二种副本同步策略会有哪些问题呢？

-- 答案：
问题1：follower同步leader的数据时，当某一个follower迟迟未向leader回复备份成功时，出现阻塞的状态；
问题2：当leader回执给producer的ack丢失时，producer因为没有收到来自leader的ack，则默认数据没有发送成功，会重新向集群发送未收到ack的消息，导致数据的重复。 -- 数据的重复指：同一条消息重复发送。


-- 那如何解决这两个问题呢？

```

- 问题1（数据阻塞）解决方案：

```sql
规则：leader完成消息的读取和写出操作，follower定时向leader拉取数据。

1. leader维护了一个动态的in-sync replicat set (ISR) 同步副本的列表，说明：即使是follower也有可能不在isr列表中。
2.。只要在isr列表中所有的follower均告知leader副本备份完成以后，则leader向producer回执ack，则不受限于出现故障的follower，因为出现故障，就被移除isr列表中。

-- 问题1：
那么什么情况下follower不在isr列表呢？
-- 答案：
如果follower没有在规定的时间与leader保持同步，则leader会将该follower从isr中踢出，同步最大时间通过replica.lag.time.max.ms参数设定。

-- 问题2：
那么从isr中踢出的follower怎么重新回到isr中呢？ --故障处理机制
-- 答案：
每个消息在follower的log文件中有：
1) 真实数据 :消息的真实数据
2) LEO(log end offset) : 消息的最后偏移量
3) HW(High Watermark) ：ISR列表中follower最小的LEO（偏移量）

说明：
1）每个follower中的LEO可能是不一样的，因副本同步的快慢有差异；
2）leader中log的LEO是最大的，因为数据源源不断的发送过来，它的落盘速度是最快的；
3）HW之前的数据对consumer可见；
4）HW是一个动态的数据，当leader回执ack一次HW就会更新一次。

follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了。

-- 问题3：
当leader挂掉以后怎么办？
-- 答案：
1） 重新选举leader；
2） 从isr列表中的follower中选取；
3） 随机选择。
'详细过程'：leader发生故障之后，会从ISR中选出一个新的leader，之后，为保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据。
'注意'：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复
```

- 问题2（数据重复）的解决方案

```sql
-- 在0.11之前的kafka版本：
在消费者端进行去重，在producer传输数据时，对消息增加唯一的全局主键，然后在消费端根据该主键进行去重。 
该方式导致消费者组所有的消费者都需要进行去重操作，重复。

-- 在0.11版本之后引进了 Exactly Once （幂等性）来解决数据重复的问题
 1） Exactly Once （幂等性） ： 做n次和做一次的效果是一样的，就是指Producer不论向Server发送多少次重复数据（重复发送同一条数据），Server端都只会持久化一条，在server端完成去重操作。
 2） 幂等性实现过程
   初始化数据时，给消息分配一个pid，发往同一个分区的消息会附带sequence Number,broker端会将<pid,partition,sequence Number>和消息的真实数据一起存储到log文件中，当具有相同主键的消息提交时，Broker只会持久化一条。

-- 何为主键？
由<pid,partition,sequence Number>三个参数构成的集合。重复发送的数据，这三个值不会变，数据是否重复与数据的内容无关，而是指为同一条数据多次发送。 -- 总结：重发的消息的主键是不会改变的，新发的消息seqnumber就会变化。

例如：消息A与消息B的数据内容完全一致
producer向集群发送消息A，集群收到以后返回的ack丢失，则消息A会被再次发送一次，此时消息A的主键是和第一次发送时相同，则集群认为数据是重复，不会进行存储；
producer向集群发送消息B，虽然与消息A的数据相同，但是seqnumber是不同的，所以不是重复的数据，集群会进行数据存储。

说明：
1） sequence Number ：消息序列号，发往同一Partition的消息会附带Sequence Number，表示该producer向该分区发送的第几次消息；
2)	pid : 生产者的id； 
3)  partition ： 分区号；
4） PID重启就会变化，同时不同的Partition也具有不同主键，所以幂等性无法保证跨分区跨会话的Exactly Once；
5） 开启幂等性会降低kafka的性能；
6） 幂等性的底层原理也还是通过给消息增加全局的唯一主键的方式；
7） 开启幂等性参数：enable.idompotence设置为true即可。
 
```

### 3.3 Kafka之消费者 consumer

#### 3.3.1 消费模式

​	消费者从server中读取数据的方式有两种：pull （拉）和 push（推）

1. pull  ： consumer向server拉取数据 **【主动】**
   - 优点：消费者按需索取
   - 缺点：不及时，有可能拉取到了空数据
2. push ：server向consumer推送数据**【被动】**
   - 优点：及时
   - 缺点：推送的速率与消费者消费的数据不一致时，产生背压

3. kafka默认使用pull，拉取数据的方式。因为kafka是一对多的关系，同一个消费者组内的不同消费者的消费速率不同，所以不好设定推送的速率。

4. 当出现拉取的数据为空时，consumer会等待一段时间之后再拉取数据，这段时长即为timeout

#### 3.3.2 分区分配策略

​	三种方式：roundrobin 、 range  、sticky

1. roundrobin ： 轮询的方式 ，理解为洗牌，一张一张的发，分区一个一个轮询的方式分配给消费者；

   缺点：当有新的消费者加进来时，所有的分区需要重新分配分区，基本上大多数的消费者的消费分区都会发生改变。

2. range：理解斗地主把牌按数量平均分配； 

   缺点：订阅的话题过多时，存在分区数量不均等的情况。

3. sticky：是在第一种方式的基础上进行改进，解决新增消费者情况的缺点，此时不再是所有消费者的分区进行重新分配，而是新进的消费者取之前所有消费者最后一次分区的数据进行消费。

- 当消费者的个数 > 分区的个数时，有些消费者没分配不到数据。
- 消费者默认的分区分配策略是range，但是消费者在消费数据时也可以自定指定策略。
- 一个分区只能由一个消费者进行消费。

#### 3.3.3   offset的维护

​       由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置的继续消费，所以consumer需要实时记录自己消费到了哪个offset，以便故障恢复后继续消费。
​      Kafka 0.9版本之前，consumer默认将offset保存在Zookeeper中，从0.9版本开始，consumer默认将offset保存在Kafka一个内置的topic中，该topic为__consumer_offsets。

```sql
-- 问题：
为什么要将offset从zookeeper中转移到kafka中？
-- 回答：
zookeeper中维护offset的效率对于Kafka来说，不可控的，Kafka不能通过修改自己的代码来提升zookeeper维护offset的效率，所以将offset的维护迁移到kafka的会话中。
```

### 3.4  Kafka高效读写数据

#### 3.4.1 顺序写磁盘

```sql
-- 问题1：
kafka的producer生产的数据最终按照顺序存储到磁盘上，写入到磁盘中数据过程不是很慢吗？
-- 回答：
1） 多分区存储模式：kafka是采用多分区的存储方式，提高了高并发；
2） 顺序写模式：按照顺序写的速度能够减少大量磁头询地址的时间，使写数据速度和网络传输速度相当，所以基本上够用，但是还是比内存数据传输的速度要慢。
```

#### 3.4.2 应用Pagecache

```sql
-- 1.说明：
Pagecache(网页缓存)：是操作系统实现的一个功能，因为linux系统兼容这个功能，所以kafka能够使用，解决大量随机读写的过程。

-- 2.内存：
我们常说的内存可以分为两个模块，一是提供给系统的内核使用，此部分对于用户是不可见的，不能被用户使用，二是供用户使用的内存。

-- 3.原理： 
pagecache是在内核内存中开辟的一个内存空间，producer生产的数据，先会存储在该内存中，待达到一定的数据量以后，再统一进行落盘，当消费者消费的速率和生产者生产的速率相同时，读写的效率是最高的，因为此时生产的数据不需要落盘处理，consumer直接从内存中读取数据。

-- 4.交换区和pagecache的区别：
交换区：将磁盘当做内存使用；
pagecache：将内存当做磁盘使用；
恰好是两个相反的过程。

-- 5.假如pagecache挂掉了怎么办？内存中的数据不是丢失了吗？
首先当发生这个问题时，是不能够完全保证数据一定不丢失，但是由于kafka具有副本策略，所以有一定保证的。

```

优点：

```sql
1） I/O Scheduler 会将连续的小块写组装成大块的物理写从而提高性能
2） I/O Scheduler 会尝试将一些写操作重新按顺序排好，从而减少磁盘头的移动时间
3） 充分利用所有空闲内存（非 JVM 内存）。如果使用应用层 Cache（即 JVM 堆内存），会增加 GC 负担
4） 读操作可直接在 Page Cache 内进行。如果消费和生产速度相当，甚至不需要通过物理磁盘（直接通过 Page Cache）交换数据
5） 如果进程重启，JVM 内的 Cache 会失效，但 Page Cache 仍然可用
```

#### 3.4.3 零拷贝技术

![图5](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200529005509.png)

说明：

内存是分级别的，读写数据时数据先经过内核内存再经过用户内存。

如果是数据的写出操作，则数据经过内核内存以后就直接往外写出，不需经过用户内存，用户内存只是负责调度的功能，减少了 数据的传输过程，这个过程称为零拷贝。

### 3.5  zookeeper在kafka中的作用

- kafka是一个去中心化的框架，没有主从之分，则需要一个中央控制中心进行调度，类似ha集群一样。
- kafka是依赖于zookeeper集群的。

流程：一个kafka集群，多个broker，一个zk集群

```sql
-- 步骤：
1） 首先所有的broker会竞选一个controller（随机竞选，谁厉害谁上），负责管理集群broker的上下线，所有topic的分区副本分配和leader选举等工作；
2） 所有的broker将自己的id信息注册到zk集群的节点上；
3） controller监控zk的这个信息；
4） controller负责broker的leader选举工作；
5） broker将状态信息注册到zk集群上；
6） 此时分区的leader故障以后，controller从zk集群中获取isr中的follower信息，负责从isr中follower选举出一个新的leader；
7） controller更新zk集群上broker的状态信息。

-- 假如故障的leader恰好也是controller怎么办？
先从现存的follower中重新选举controller，再执行1-5步。
```

![图6](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200529005514.png)

### 3.6 Kafka事务

```sql
--问题：
事务用来解决什么问题？
--回答：
kafka使用Exactly Once解决producer端生产数据重复的问题存在什么问题？

问题1：不能跨分区；
问题2：producer重启时，pid会发生变化。

则事务就是来解决上面问题的，事务可以保证Kafka在Exactly Once语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。

-- 那具体是怎么做的呢？
```

#### 3.6.1 producer事务

```sql
--解决producer重启问题：
1) 引进全局唯一的Transaction ID，将producer的pid与Transaction ID进行绑定。当重启producer时，可以通过正在进行的Transaction ID获得原来的PID.
2) 为了管理Transaction，Kafka引入了一个新的组件Transaction Coordinator。Producer就是通过和Transaction Coordinator交互获得Transaction ID对应的任务状态。Transaction Coordinator还负责将所有事务写入Kafka的一个内部Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。
```

#### 3.6.2  Consumer事务（精准一次性消费）

```
kafak对consumer事务的保证是非常弱的，尤其无法保证Commit的信息被精确消费。这是由于Consumer可以通过offset访问任意信息，而且不同的Segment File生命周期不同，同一事务的消息可能会出现重启后被删除的情况
```

## 四、 Kafka API

温馨提示

```sql
api的步骤：

第一步： new 对象;
第二步： 具体的操作;
第三步： 关闭资源。

-- 不知道要写哪些参数？不知道参数的意义？不知道参数取值？怎么办？
请认准kafka官网：https://kafka.apache.org/documentation/
producer API ： 找Producer Configs
consumer API：  找Consumer Configs
```

### 4.1 Producer API

#### 4.1.1 消息发送流程

```sql
-- 问题： 
Kafka的Producer发送消息采用的是异步发送的方式，这种方式优点和缺点是什么呢？
-- 回答：
优点：高效率，生产者只要一直生产数据就可以，不需要等到ack回执后再进行生产数据；
缺点：不能实时知道数据是否发送成功，不过有ack机制、幂等性机制和producer事务（保证数据的准确性）。
```

```sql
-- 发送数据的流程：
'两线程一共享变量'：
1. main线程：将消息发送给RecordAccumulator
2. Sender线程：Sender线程不断从RecordAccumulator中拉取消息发送到Kafka broker
3. 线程共享变量——RecordAccumulator：数据临时存储器。
'步骤'
第一步：生产者首先将数据包装成ProducerRecord
第二步：main线程中有一个send方法，producer将ProducerRecord发送给interceptors'拦截器'处理；
第三步：interceptors处理好后将数据传递给'序列化器'，将数据序列化； -- 在producer端序列化
第四步：将序列化好的数据传递给'分区器'，对数据进行分区； -- 在producer端序列化
第五步：将数据传递到内存的数据缓存区，在这里面，话题有多少个分区，在缓存区里面就有多少个分区，一一对应，对应的分区数据就会去到对应的缓存区的分区中； -- 此时的数据是已经分好区了，同时也是已经序列化，此时producer就不再管这里的数据了；
第六步：Sender线程就将数据发送给topic中的分区中。 

-- 此时的数据，Sender线程是怎么向topic中发的呢？
batch.size：只有数据积累到batch.size之后，sender才会发送数据。
linger.ms：如果数据迟迟未达到batch.size，sender等待linger.time之后就会发送数据。

```

![图7](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200529005518.png)

#### 4.1.2 异步发送API

```java
package kafkaproducer;


import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

/**
 * @author lianzhipeng
 * @Description
 * @create 2020-05-08 14:58
 */

public class Producer {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        //1.new 对象

        Properties properties = new Properties();
        properties.setProperty("key.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");
        properties.setProperty("value.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");
        properties.setProperty("acks", "all");
        properties.setProperty("bootstrap.servers", "hadoop102:9092");

        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        //2.具体的操作
        for (int i = 0; i < 100; i++) {
            Future<RecordMetadata> result = producer.send(new ProducerRecord<String, String>(
                    "first",
                    "Message" + i,
                    "这是第" + i + "条信息"
            ), new Callback() {//回调函数，当producer发送的数据完成以后，返回告诉producer数据发送成功
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    int partition = metadata.partition();
                    String topic = metadata.topic();
                    long offset = metadata.offset();
                    System.out.println(
                            topic + "话题"
                                    + partition + "分区"
                                    + offset + "消息发送成功");
                }
            });
            /*
           
            如下一行代码产生同步回调和异同回调两种方式：
            同步回调：加了此行代码，生产者收到ack以后再发第二条消息；类似打电话
            异步回调：未加此行代码，生成者只要一直发送消息既可。类似发短信
          
            */
            RecordMetadata recordMetadata = result.get();

            System.out.println("第" + i + "条消息发送结束");

        }

        //3.关闭资源，资源关闭的时候会调用回调函数
        producer.close();

    }
}

```

### 4.2  Consumer API

#### 4.2.1 数据漏消费和重复消费

1. 消费者不用担心数据的可靠性问题，因为消费者消费以后的数据是不会从kafka集群中删除的。但是消费者要关心两个问题：

```sql
-- 问题1 数据漏消费
什么时候会出现数据漏消费呢？
先提交offset后消费。
例如：消费者从kafka集群中获取了数据，此数据在消费的过程中出现故障延迟最后宕机，在故障期间offset已经提交至kafka集群，此时实际上数据并没有被使用，但是kafka集群上该消费者消费的数据偏移量已经更新了，重启消费者时，上一条数据不能被消费了，导致数据漏消费。
-- 问题2 数据重复消费
什么时候会出现数据库重复消费呢？
当数据已经被消费以后，此时返回的offset时消费者出现了故障，则kafka集群中的_consumer_offset会话保存的offset则为上一次的数据，offset没有被更新，当消费者重新启动时，上一条数据则会被重新再消费一次。
```

2. 谈谈消费者提交offset的模式

  消费者每次拉取数据的最大值为：1M，（ 1048576字节）

- 模式一：自动提交，默认每5s提交一次；

- 模式二：手动提交，两种方式：commitSync（同步提交）、commitAsync（异步提交）；

​        同步和异步的异同点：

```sql
-- 相同点：
提交本次poll的一批数据最高的偏移量.
-- 不同点：
commitSync（同步提交）：提交offset时，commitSync阻塞当前线程，一直到提交成功，并且会自动失败重试（由不可控因素导致，也会出现提交失败）；
commitAsync（异步提交）：则没有失败重试机制，故有可能提交失败。
```

#### 4.2.2 几个重要的参数

1. 自动提交offset的时间：默认为5s

[auto.commit.interval.ms](https://kafka.apache.org/documentation/#auto.commit.interval.ms)

| Type:         | int     |
| ------------- | ------- |
| Default:      | 5000    |
| Valid Values: | [0,...] |
| Importance:   | low     |

2. 消费者消费数据的起始位置

[auto.offset.reset](https://kafka.apache.org/documentation/#auto.offset.reset)

- earliest: automatically reset the offset to the earliest offset -->表示消费topic所有的数据
- latest: automatically reset the offset to the latest offset  -->表示只消费最新的数据

| Type:         | string                   |
| ------------- | ------------------------ |
| Default:      | latest                   |
| Valid Values: | [latest, earliest, none] |
| Importance:   | medium                   |

3. 一次从一个分区拉取的最大数据量

[max.partition.fetch.bytes](https://kafka.apache.org/documentation/#max.partition.fetch.bytes)

| Type:         | int     |
| ------------- | ------- |
| Default:      | 1048576 |
| Valid Values: | [0,...] |
| Importance:   | high    |

#### 4.2.3 代码

- ### Consumer API

```java
package kafkaconsumer;

import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;

import java.time.Duration;
import java.util.Collections;
import java.util.Map;
import java.util.Properties;

/**
 * @author lianzhipeng
 * @Description
 * @create 2020-05-08 21:04
 */

public class MyConsumer {
    public static void main(String[] args) {


        //1 new 对象

        Properties properties = new Properties();

        properties.setProperty("key.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("bootstrap.servers", "hadoop102:9092");
        properties.setProperty("group.id", "group9");
        properties.setProperty("auto.offset.reset", "earliest");
        //自动提交offset
        properties.setProperty("enable.auto.commit","false");


        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);


        //2 操作
        //连接话题
        consumer.subscribe(Collections.singleton("first"));
        //拉取数据
        Duration duration = Duration.ofMillis(500);
        while (true){
            ConsumerRecords<String, String> records = consumer.poll(duration);
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(record);
            }
            //手动同步提交
//            consumer.commitSync();
            //手动异步提交
            consumer.commitAsync(new OffsetCommitCallback() {
                @Override
                public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
                    offsets.forEach(
                            (t, o) -> {
                                System.out.println("分区：" + t + "\nOffset：" + o);
                            }
                    );
                }
            });
        }
        //3 关闭资源
//        consumer.close();
    }

}

```

- 异步提交代码：

```java
//手动异步提交方式，形参里面为回调对象。
            consumer.commitAsync(new OffsetCommitCallback() {
                /*
               回调方式，当消费成功以后调用此方法并进行打印
                */
                @Override
                public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
                    offsets.forEach(
                            (t, o) -> {
                                System.out.println("分区：" + t + "\nOffset：" + o);
                            }
                    );
                }
            });
```

## 五、Kafka监控（Kafka Eagle）

1. 修改kafka启动命令

```sql
修改kafka-server-start.sh命令中

--原文：
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
fi

--改为：
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-server -Xms2G -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"
    export JMX_PORT="9999"
    #export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
fi

--注意：修改之后在启动Kafka之前要分发之其他节点

```

2. 上传压缩包kafka-eagle-bin-1.3.7.tar.gz到集群/opt/software目录
3. 解压到本地

```sql
[atguigu@hadoop102 software]$ tar -zxvf kafka-eagle-bin-1.3.7.tar.gz
```

4. 进入刚才解压的目录,将kafka-eagle-web-1.3.7-bin.tar.gz解压至opt/module

```sql
[atguigu@hadoop102 kafka-eagle-bin-1.3.7] $ tar -zxvf kafka-eagle-web-1.4.5-bin.tar.gz -C /opt/module/
```

5. 修改名称

```
[atguigu@hadoop102 module]$ mv kafka-eagle-we-1.4.5/   eagle
```

6. 给启动文件执行权限 /opt/module/eagle/bin

```sql
[atguigu@hadoop102 bin]$ chmod 777 ke.sh
```

7. 修改配置文件 /opt/module/eagle/conf/system-config.properties

```sql
######################################
# multi zookeeper&kafka cluster list
######################################
kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka


######################################
# kafka offset storage
######################################
cluster1.kafka.eagle.offset.storage=kafka


######################################
# kafka metrics, 30 days by default
######################################
kafka.eagle.metrics.charts=true
kafka.eagle.metrics.retain=30


######################################
# kafka sqlite jdbc driver address

######################################
kafka.eagle.driver=com.mysql.jdbc.Driver
kafka.eagle.url=jdbc:mysql://hadoop102:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
kafka.eagle.username=root
kafka.eagle.password=123456
```

8. 添加环境变量

```sql
export KE_HOME=/opt/module/eagle
export PATH=$PATH:$KE_HOME/bin
-- 注意：source /etc/profile
```

9. 启动

```sql
[atguigu@hadoop102 eagle]$ bin/ke.sh start

... ...
... ...
*******************************************************************
* Kafka Eagle Service has started success.
* Welcome, Now you can visit 'http://192.168.9.102:8048/ke' -- 这个网址就是登入的eagle的网址
* Account:admin ,Password:123456 -- 这是登入的密码
*******************************************************************
* <Usage> ke.sh [start|status|stop|restart|stats] </Usage>
* <Usage> https://www.kafka-eagle.org/ </Usage>
*******************************************************************

--注意：启动之前需要先启动ZK以及KAFKA
```

10. 登录页面查看监控数据

```sql
网址： http://192.168.9.102:8048/ke
账号： admin
密码： 123456
```

## 六、面试题

### 6.1  Kafka中的ISR、AR代表什么

```sql
ISR:分区leader维护的一个follower列表，在isr中的follower与leader同步。
AR:分区的所有副本。
```

### 6.2 Kafka中的HW、LEO等分别代表什么

```sql
LEO: leader维护的isr中所有follower的最后偏移量。
HW：所有followerleo最小的值。
```

### 6.3 Kafka中是怎么体现消息顺序性的

```sql
每次生产的数据是在一个上次生产数据的基础上追加，同时存储了消息的offset和数据的index索引，减少了数据存储时的磁头寻址的过程。
```

### 6.4 Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？

```sql
处理顺序： 拦截器 --> 序列化器 --> 分区器
拦截器：对数据进行简单处理，加一些标识。
序列化：对数据进行序列化，保证数据可用于传输；
分区器：给数据加上分区标签，指定数据应该去到哪个kafka集群中的分区。

以上三步骤均在producer端就完成了。

```

### 6.5  Kafka生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？

```
一共2个线程，一个数据缓存区。
线程
main线程：负责对数据进行包装、序列化、分区。
sender线程：负责将数据从数据缓冲区发送topic话题中。

```

### 6.6 消费者组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据这句话是否正确

```
是的，正确。
了解一下分区分配的策略。
三种方式：roundrobin 、 range  、sticky。
```

### 6.7 消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1？

```
offset + 1 。
```

### 6.8 有哪些情形会造成重复消费？

```
先消费后提交offset。
```

### 6.9 那些情景会造成消息漏消费？

```
先提交offset后消费。
```

### 6.10 当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？

```
了解producer发送数据的过程。
```

### 6.11 topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？

```
可以增加。
```

### 6.12 topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？

```
不能减少，因为原分区中的数据没有地方去。
```

### 6.13 Kafka有内部的topic吗？如果有是什么？有什么作用？

```

会话：_consumer_offset，保存consumer消费的偏移量。
```

### 6.14 Kafka分区分配的概念？

```sql
一共有三种分区分配的策略。
三种方式：
1）roundrobin ： 轮询分配。
2）range ： 平均分配。
3）sticky ： 轮询分配 + 解决新增消费者的优化。
```

### 6.15 简述Kafka的日志目录结构？

```sql
一共有3个文件
1）log文件：记录真实数据，内部包含了真实数据 + hw + leo。
2）index文件 ： 存储消息的偏移量。
3) timeindex文件 ： 存储下消息的时间偏移量。
```

### 6.16 如果我指定了一个offset，Kafka Controller怎么查找到对应的消息？

```
通过offset，消息的偏移量，通过日志目录的文件顺序号，根据区间范围找到消息所在的inde和log目录。
其次根据在index表中的消息偏移量找到真实数据在log文件中该消息的起始索引位置。
```

### 6.17聊一聊Kafka Controller的作用？

```
1、负责leader的选举；
2、负责监控leader的状态；
3.负责更新集群在zookeeper中的状态。
```

### 6.18  Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？

```
1.每个分区的leader选举；(isr)；
2.controller的选举（先到先得）。
```

### 6.19  失效副本是指什么？有那些应对措施？

```
follower不能与leader进行同步数据，暂时被leader踢出isr列表中。通过followe故障恢复重新备份，当leo达到了isr中的hw时，又重新会回到isr的列表中。
```

### 6.20  Kafka的那些设计让它有如此高的性能？

```
1. pagecache；
2.顺序读写机制；
3.零拷贝技术；
4.多分区策略。
```

## 七 、flume与kafka融合技术

kafka：数据的中转站，主要功能由topic体现；

flume：数据的采集，通过source和sink体现。

### 7.1 kafka source

```sql
-- 问题 ：
fulme在kafka中的作用
-- 答案：
消费者
```



```sql
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource --source类型
a1.sources.r1.kafka.bootstrap.servers = hadoop105:9092,hadoop106:9092 -- kafka的集群
a1.sources.r1.kafka.topics=topic_log -- 订阅的话题
a1.sources.r1.batchSize=6000 --putlist中数据达到了6K以后提交到channel中
a1.sources.r1.batchDurationMillis=2000 --拉取数据的时间达到2s以后，将获取的数据提交到channel中
```



### 7.2 kakfa channel

- kakfa channel这种情况使用的最多，此时的flume可以是消费者、生产者、source和sink之间的缓冲区（具有高吞吐量的优势），Channel是位于Source和Sink之间的缓冲区。
- 一共有三种情况，分别是:

```sql
-- 情况一： 有Flume source and sink -- 缓冲区
kakfa channel为事件提供了可靠且高可用的通道；

-- 情况二： 有source and interceptor but no sink --生产者
it allows writing Flume events into a Kafka topic, for use by other app

-- 情况三： 有 sink, but no source --消费者
it is a low-latency, fault tolerant way to send events from Kafka to Flume sinks such as HDFS, HBase or Solr
```

官方配置文件：

```sql
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel ----channel类型
a1.channels.c1.kafka.bootstrap.servers = hadoop105:9092,hadoop106:9092,hadoop107:9092 --kafka集群
a1.channels.c1.kafka.topic =topic_log --话题
a1.channels.c1.parseAsFlumeEvent=false --不需要event的header数据
```

### 7.3 kafka sink

作用：将数据拉去到kafka的topic中。

```sql
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink --sink类型
a1.sinks.k1.kafka.topic =topic_log --话题
a1.sinks.k1.kafka.bootstrap.servers = hadoop105:9092,hadoop106:9092,hadoop107:9092 --kafka集群
a1.sinks.k1.kafka.flumeBatchSize = 20
a1.sinks.k1.kafka.producer.acks = 1 --副本策略
a1.sinks.k1.kafka.producer.linger.ms = 1 
a1.sinks.k1.kafka.producer.compression.type = snappy  --压缩格式
```



