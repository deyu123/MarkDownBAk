# Flume练习题

*************

## 题目1

需求：使用Flume监听一个端口，收集该端口数据，并打印到控制台。 



```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost 
a1.sources.r1.port = 44444  

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 
#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = logger
#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1

```

## 题目2

实时监控Hive日志，并上传到HDFS中

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1
#步骤二：source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/hive/logs/hive.log 
a1.sources.r1.shell = /bin/bash -c

#步骤三： channel selector
a1.sources.r1.selector.type = replicating
#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 
#步骤五： sinkprocessor，默认配置defaultsinkprocessor
#步骤六： sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/upload2/%Y%m%d/%H   
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 60 
#设置每个文件的滚动大小大概是128M
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0
#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## 题目3

使用Flume监听整个目录的文件，并上传至HDFS

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1
#步骤二：source
# Describe/configure the source
a1.sources.r1.type = spooldir 
a1.sources.r1.spoolDir = /opt/module/flume/upload 
a1.sources.r1.fileSuffix = .COMPLETED 
a1.sources.r1.fileHeader = true 
#忽略所有以.tmp结尾的文件，不上传
a1.sources.r1.ignorePattern = ([^ ]*\.tmp)

#步骤三： channel selector
a1.sources.r1.selector.type = replicating
#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 
#步骤五： sinkprocessor，默认配置defaultsinkprocessor
#步骤六： sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/upload/%Y%m%d/%H   
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 60 
#设置每个文件的滚动大小大概是128M
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0
#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## 题目4

使用Flume监听整个目录的实时追加文件，并上传至HDFS

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1
#步骤二：source
# Describe/configure the source
a1.sources.r1.type = TAILDIR 
a1.sources.r1.positionFile = /opt/module/flume/tail_dir.json -- 指定position_file 的位置(记录每次上传后的偏移量，实现断点续传的关键)
a1.sources.r1.filegroups = f1 f2 -- 监控的文件目录集合
a1.sources.r1.filegroups.f1 = /opt/module/flume/files/.*file.* -- 定义监控的文件目录1
a1.sources.r1.filegroups.f2 = /opt/module/flume/files/.*log.* -- 定义监控的文件目录2

#步骤三： channel selector
a1.sources.r1.selector.type = replicating
#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 
#步骤五： sinkprocessor，默认配置defaultsinkprocessor
#步骤六： sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/upload3/%Y%m%d/%H   
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 60 
#设置每个文件的滚动大小大概是128M
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0
#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## 题目5

使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3负责输出到Local FileSystem。

- flume1：

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2
#步骤二：source
# Describe/configure the source
a1.sources.r1.type = TAILDIR 
a1.sources.r1.positionFile = /opt/module/flume/tail_dir.json 
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/flume/files/.*log.* 
#步骤三： channel selector
a1.sources.r1.selector.type = replicating
#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000 
a1.channels.c2.transactionCapacity = 100 
#步骤五： sinkprocessor，默认配置defaultsinkprocessor
#步骤六： sink
# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop103 
a1.sinks.k1.port = 6666  

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop104 
a1.sinks.k2.port = 8888  

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```

- flume2:

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1
#步骤二：source
# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop103 
a1.sources.r1.port = 6666 
#步骤三： channel selector
a1.sources.r1.selector.type = replicating
#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor
#步骤六： sink
# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/upload4/%Y%m%d/%H   
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 60  
#设置每个文件的滚动大小大概是128M
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

- flum3:

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop104 
a1.sources.r1.port = 8888

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = file_roll
a1.sinks.k1.sink.directory = /opt/module/flume/datas/flume3

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## 题目6

使用Flume1监控一个端口，其sink组中的sink分别对接Flume2和Flume3，采用FailoverSinkProcessor，实现故障转移的功能

- flume1

```sql
#步骤一：agent Name
a1.sources = r1
a1.channels = c1
a1.sinkgroups = g1
a1.sinks = k1 k2

#步骤二：source
# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost 
a1.sources.r1.port = 44444 

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor
a1.sinkgroups.g1.processor.type = failover 
a1.sinkgroups.g1.processor.priority.k1 = 10 
a1.sinkgroups.g1.processor.priority.k2 = 5 
a1.sinkgroups.g1.processor.maxpenalty = 10000 

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop103
a1.sinks.k1.port = 1111  

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop104
a1.sinks.k2.port = 2222  

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c1
```

- flume2

```Sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop103
a1.sources.r1.port = 1111 

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = logger 

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

- flume3

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
# Describe/configure the source
a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop104
a1.sources.r1.port = 2222

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = logger 

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

## 题目7

hadoop102上的Flume-1监控文件/opt/module/group.log，

hadoop103上的Flume-2监控某一个端口的数据流，

Flume-1与Flume-2将数据发送给hadoop104上的Flume-3，Flume-3将最终数据打印到控制台。

- flume1

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
a1.sources.r1.type = TAILDIR 
a1.sources.r1.positionFile = /opt/module/flume/tail_dir.json 
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/flume/files/.*log.* 

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop104
a1.sinks.k1.port = 4141  

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

- flume2

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost 
a1.sources.r1.port = 3333

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop104
a1.sinks.k1.port = 4141  

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

- flume3

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop104
a1.sources.r1.port = 4141 

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = logger 

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```



## 题目8

需求：

1. a1 102 接收TailDirSource数据，监控/var/log/*.log，复制输出到a2 a3

2. a2 103 接收a1数据，输出到HDFS，failover到本地FileRoll
3. a3 104 接收a1数据，输出到控制台

- flume1

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1 k2
a1.channels = c1 c2

#步骤二：source

a1.sources.r1.type = TAILDIR 
a1.sources.r1.positionFile = /opt/module/flume/tail_dir.json 
a1.sources.r1.filegroups = f1 
a1.sources.r1.filegroups.f1 = /opt/module/flume/files/.*file.* 

#步骤三： channel selector
a1.sources.r1.selector.type = replicating


#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

a1.channels.c2.type = memory
a1.channels.c2.capacity = 1000 
a1.channels.c2.transactionCapacity = 100

#步骤五： sinkprocessor，默认配置defaultsinkprocessor


#步骤六： sink
# Describe the sink
a1.sinks.k1.type = avro
a1.sinks.k1.hostname = hadoop103
a1.sinks.k1.port = 6666

a1.sinks.k2.type = avro
a1.sinks.k2.hostname = hadoop104 
a1.sinks.k2.port = 8888

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```

- flume2:

```sql
#步骤一：agent Name
a1.sources = r1
a1.channels = c1
a1.sinkgroups = g1
a1.sinks = k1 k2

#步骤二：source
a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop103
a1.sources.r1.port = 6666 

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100 

#步骤五： sinkprocessor，默认配置defaultsinkprocessor
a1.sinkgroups.g1.processor.type = failover 
a1.sinkgroups.g1.processor.priority.k1 = 10
a1.sinkgroups.g1.processor.priority.k2 = 5 
a1.sinkgroups.g1.processor.maxpenalty = 10000 

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop102:9820/flume/upload2/%Y%m%d/%H  
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix = upload-
#是否按照时间滚动文件夹
a1.sinks.k1.hdfs.round = true
#多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
#重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
#是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型，可支持压缩
a1.sinks.k1.hdfs.fileType = DataStream
#多久生成一个新的文件
a1.sinks.k1.hdfs.rollInterval = 60  
#设置每个文件的滚动大小大概是128M
a1.sinks.k1.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0

a1.sinks.k2.type = logger

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c1
```

- flume3:

```sql
#步骤一：agent Name
a1.sources = r1
a1.sinks = k1
a1.channels = c1

#步骤二：source
a1.sources.r1.type = avro
a1.sources.r1.bind = hadoop104
a1.sources.r1.port = 8888

#步骤三： channel selector
a1.sources.r1.selector.type = replicating

#步骤四： channel
# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100

#步骤五： sinkprocessor，默认配置defaultsinkprocessor

#步骤六： sink
# Describe the sink
a1.sinks.k1.type = logger

#步骤七：连接source、channel、sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

