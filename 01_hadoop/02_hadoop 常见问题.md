# Hadoop 常见问题

### 1.常见问题

```shell
-- Hadoop 需要启动哪些进程

1）NameNode它是hadoop中的主服务器，管理文件系统名称空间和对集群中存储的文件的访问，保存有metadate。
2）SecondaryNameNode它不是namenode的冗余守护进程，而是提供周期检查点和清理任务。帮助NN合并editslog，减少NN启动时间。
3）DataNode它负责管理连接到节点的存储（一个集群中可以有多个节点）。每个存储数据的节点运行一个datanode守护进程。
4）ResourceManager（JobTracker）JobTracker负责调度DataNode上的工作。每个DataNode有一个TaskTracker，它们执行实际工作。
5）NodeManager（TaskTracker）执行任务。
6）DFSZKFailoverController高可用时它负责监控NN的状态，并及时的把状态信息写入ZK。它通过一个独立线程周期性的调用NN上的一个特定接口来获取NN的健康状态。FC也有选择谁作为Active NN的权利，因为最多只有两个节点，目前选择策略还比较简单（先到先得，轮换）。
7）JournalNode 高可用情况下存放namenode的editlog文件。

-- 简述Hadoop的几个默认端口及其含义。
1）dfs.namenode.http-address:50070
2）SecondaryNameNode辅助名称节点端口号：50090
3）dfs.datanode.address:50010
4）fs.defaultFS:8020 或者9000
5）yarn.resourcemanager.webapp.address:8088


-- 服役新数据节点和退役旧节点步骤
1）节点上线操作：
当要新上线数据节点的时候，需要把数据节点的名字追加在 dfs.hosts 文件中
（1）关闭新增节点的防火墙
（2）在 NameNode 节点的 hosts 文件中加入新增数据节点的 hostname
（3）在每个新增数据节点的 hosts 文件中加入 NameNode 的 hostname
（4）在 NameNode 节点上增加新增节点的 SSH 免密码登录的操作
（5）在 NameNode 节点上的 dfs.hosts 中追加上新增节点的 hostname,
（6）在其他节点上执行刷新操作：hdfs dfsadmin -refreshNodes
（7）在 NameNode 节点上，更改 slaves 文件，将要上线的数据节点 hostname 追加到 slaves 文件中
（8）启动 DataNode 节点
（9）查看 NameNode 的监控页面看是否有新增加的节点
2）节点下线操作：
（1）修改/conf/hdfs-site.xml 文件
（2）确定需要下线的机器，dfs.osts.exclude 文件中配置好需要下架的机器，这个是阻止下架的机器去连接 NameNode。
（3）配置完成之后进行配置的刷新操作./bin/hadoop dfsadmin -refreshNodes,这个操作的作用是在后台进行 block 块的移动。
（4）当执行三的命令完成之后，需要下架的机器就可以关闭了，可以查看现在集群上连接的节点，正在执行 Decommission，会显示：Decommission Status : Decommission in progress 执行完毕后，会显示：Decommission Status : Decommissioned
（5）机器下线完毕，将他们从excludes 文件中移除。

-- Namenode挂了怎么办？
方法一：将SecondaryNameNode中数据拷贝到namenode存储数据的目录；
方法二：使用-importCheckpoint选项启动namenode守护进程，从而将SecondaryNameNode中数据拷贝到namenode目录中。


-- 谈谈Hadoop序列化和反序列化及自定义bean对象实现序列化?
1）序列化和反序列化
序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输。 
反序列化就是将收到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。
Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系等），不便于在网络中高效传输。所以，hadoop自己开发了一套序列化机制（Writable），精简、高效。
2）自定义bean对象要想序列化传输步骤及注意事项：。
（1）必须实现Writable接口
（2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造
（3）重写序列化方法
（4）重写反序列化方法
（5）注意反序列化的顺序和序列化的顺序完全一致
（6）要想把结果显示在文件中，需要重写toString()，且用”\t”分开，方便后续用
（7）如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序

2.FileInputFormat切片机制
（1）简单地按照文件的内容长度进行切片
（2）切片大小，默认等于block大小
（3）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片
3.自定义InputFormat流程
（1）自定义一个类继承FileInputFormat
（2）改写RecordReader，实现一次读取一个完整文件封装为KV
4.如何决定一个job的map和reduce的数量?
1）map数量
splitSize=max{minSize,min{maxSize,blockSize}}
map数量由处理的数据分成的block数量决定default_num = total_size / split_size;
2）reduce数量
reduce的数量job.setNumReduceTasks(x);x 为reduce的数量。不设置的话默认为 1。
5.Maptask的个数由什么决定？
一个job的map阶段MapTask并行度（个数），由客户端提交job时的切片个数决定。
```

### 2.MapTask 工作机制

![image-20201207222646142](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201207222646142.png)

```shell
（1）Read阶段：Map Task通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
	（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
	（3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
	（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。
	溢写阶段详情：
	步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。
	步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
	步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。
	（5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。
	当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。
	在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认100）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。
	让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。
```

### 3.ReduceTask 阶段

![image-20201207223809259](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201207223809259.png)

```
（1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
（2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。
（3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。
（4）Reduce阶段：reduce()函数将计算结果写到HDFS上
```

###  4. MapReduce 有几种排序及排序发生的阶段

```shell
1）排序的分类：
	（1）部分排序：
MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部排序。
	（2）全排序：
如何用Hadoop产生一个全局排序的文件？最简单的方法是使用一个分区。但该方法在处理大型文件时效率极低，因为一台机器必须处理所有输出文件，从而完全丧失了MapReduce所提供的并行架构。
	替代方案：首先创建一系列排好序的文件；其次，串联这些文件；最后，生成一个全局排序的文件。主要思路是使用一个分区来描述输出的全局排序。例如：可以为待分析文件创建3个分区，在第一分区中，记录的单词首字母a-g，第二分区记录单词首字母h-n, 第三分区记录单词首字母o-z。
（3）辅助排序：（GroupingComparator分组）
	Mapreduce框架在记录到达reducer之前按键对记录排序，但键所对应的值并没有被排序。甚至在不同的执行轮次中，这些值的排序也不固定，因为它们来自不同的map任务且这些map任务在不同轮次中完成时间各不相同。一般来说，大多数MapReduce程序会避免让reduce函数依赖于值的排序。但是，有时也需要通过特定的方法对键进行排序和分组等以实现对值的排序。
	（4）二次排序：
	在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。
2）自定义排序WritableComparable
bean对象实现WritableComparable接口重写compareTo方法，就可以实现排序
@Override
public int compareTo(FlowBean o) {
	// 倒序排列，从大到小
	return this.sumFlow > o.getSumFlow() ? -1 : 1;
}
3）排序发生的阶段：
（1）一个是在map side发生在spill后partition前。
（2）一个是在reduce side发生在copy后 reduce前。


12.如果没有定义partitioner，那数据在被送达reducer前是如何被分区的？
如果没有自定义的 partitioning，则默认的 partition 算法，即根据每一条数据的 key
的 hashcode 值摸运算（%）reduce 的数量，得到的数字就是“分区号”。

13.MapReduce 怎么实现 TopN？
可以自定义groupingcomparator，或者在map端对数据进行排序，然后再reduce输出时，控制只输出前n个数。就达到了topn输出的目的。

14.有可能使 Hadoop 任务输出到多个目录中么？如果可以，怎么做？
1）可以输出到多个目录中，采用自定义OutputFormat。
2）实现步骤：
（1）自定义outputformat，
（2）改写recordwriter，具体改写输出数据的方法write()


```

### 5. Hadoop 实现Join 的方法及方法的实现？

```shell
1）reduce side join
Map端的主要工作：为来自不同表(文件)的key/value对打标签以区别不同来源的记录。然后用连接字段作为key，其余部分和新加的标志作为value，最后进行输出。
Reduce端的主要工作：在reduce端以连接字段作为key的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录(在map阶段已经打标志)分开，最后进行合并就ok了。
2）map join
在map端缓存多张表，提前处理业务逻辑，这样增加map端业务，减少reduce端数据的压力，尽可能的减少数据倾斜。
具体办法：采用distributedcache
		（1）在mapper的setup阶段，将文件读取到缓存集合中。
		（2）在驱动函数中加载缓存。
job.addCacheFile(new URI("file:/e:/mapjoincache/pd.txt"));// 缓存普通文件到task运行节点


17.参考下面的MR系统的场景:
--hdfs块的大小为128MB
--输入类型为FileInputFormat
--有三个文件的大小分别是:64KB 130MB 260MB
Hadoop框架会把这些文件拆分为多少块？
4块：64K，130M，128M，132M


```

### 6. Hadoop 中RecordReader 的作用？

```shell
（1）以怎样的方式从分片中读取一条记录，每读取一条记录都会调用RecordReader类；
（2）系统默认的RecordReader是LineRecordReader
（3）LineRecordReader是用每行的偏移量作为map的key，每行的内容作为map的value；
（4）应用场景：自定义读取每一条记录的方式；自定义读入key的类型，如希望读取的key是文件的路径或名字而不是该行在文件中的偏移量。

```

### 7. HDFS 的压缩算法？及运用场景

![img](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/v2-78587df4f63ee2501a403045ecabd4bd_720w.jpg)

```shell
1）gzip压缩
优点：压缩率比较高，而且压缩/解压速度也比较快；hadoop本身支持，在应用中处理gzip格式的文件就和直接处理文本一样；大部分linux系统都自带gzip命令，使用方便。
缺点：不支持split。
应用场景：当每个文件压缩之后在130M以内的（1个块大小内），都可以考虑用gzip压缩格式。例如说一天或者一个小时的日志压缩成一个gzip文件，运行mapreduce程序的时候通过多个gzip文件达到并发。hive程序，streaming程序，和java写的mapreduce程序完全和文本处理一样，压缩之后原来的程序不需要做任何修改。
	2）Bzip2压缩
优点：支持split；具有很高的压缩率，比gzip压缩率都高；hadoop本身支持，但不支持native；在linux系统下自带bzip2命令，使用方便。
缺点：压缩/解压速度慢；不支持native。
应用场景：适合对速度要求不高，但需要较高的压缩率的时候，可以作为mapreduce作业的输出格式；或者输出之后的数据比较大，处理之后的数据需要压缩存档减少磁盘空间并且以后数据用得比较少的情况；或者对单个很大的文本文件想压缩减少存储空间，同时又需要支持split，而且兼容之前的应用程序（即应用程序不需要修改）的情况。
	3）Lzo压缩
优点：压缩/解压速度也比较快，合理的压缩率；支持split，是hadoop中最流行的压缩格式；可以在linux系统下安装lzop命令，使用方便。
缺点：压缩率比gzip要低一些；hadoop本身不支持，需要安装；在应用中对lzo格式的文件需要做一些特殊处理（为了支持split需要建索引，还需要指定inputformat为lzo格式）。
应用场景：一个很大的文本文件，压缩之后还大于200M以上的可以考虑，而且单个文件越大，lzo优点越越明显。
	4）Snappy压缩
优点：高速压缩速度和合理的压缩率。
缺点：不支持split；压缩率比gzip要低；hadoop本身不支持，需要安装； 
应用场景：当Mapreduce作业的Map输出的数据比较大的时候，作为Map到Reduce的中间数据的压缩格式；或者作为一个Mapreduce作业的输出和另外一个Mapreduce作业的输入。
```

