# Redis

***

## 一、Redis入门

### 1.1 redis介绍

```sql
-- 1. redis是什么？
   	  remote dictionary server ： 远程字典服务器，是一种'基于内存'的数据结构存储系统,所以对于内存要求很高。
   	  字典：就是map
   	  使用c语言写的，非常轻巧。
-- 2. 能用来干什么？
       1. 作为数据库	
       2. 作为缓存
       3. 作为消息的中间件，kafka也是消息中间件
       我们主要是用来做缓存，客户端访问数据时，不用直接访问mysql数据，而是访问redis，提高响应的速度，同时缓解mysql的压力。
-- 3. 数据结构说明：
       1. 是一个kv键值对的数据结构，nosql
       2. k固定为string类型，v的类型有很多，我们主要学习如下5大数据类型
          1. 字符串
          2. list：有序可重复
          3. set：无序不重复，可用来对value进行去重，'作用非常大'
          4. hash：类型map<string,string>
          5. sorted set = zset ，带分数的set，排序后的set
       3. v不支持嵌套，所以list、set、map中的数据都是string
```

![image-20200710191434441](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710191435.png)

![image-20200723095400289](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200723095407.png)

### 1.2 安装

> 说明：安装和启动redis均使用root用户

```sql
第一步： 将 redis-3.2.5.tar.gz 上传到 opt/sofeware
第二步： 解压到opt/module
```

- 第三步： 安装gcc环境：我们需要将源码编译后再安装，因此需要安装c语言的编译环境！不能直接make！

```bash
   yum install –y gcc-c++
```

- 第四步：查看安装是否成功

```
rpm –qa|grep gcc
```

![image-20200710193135816](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710193413.png)

```sql
-- 此时安装的常见错误
   错误1： 在没有安装gcc环境下，如果执行了make，不会成功！安装环境后，第二次make有可能报错
     		'Jemalloc/jemalloc.h:没有那个文件'
   解决方案：运行 make distclean之后再make
```

![image-20200710193408711](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710193408.png)

- 第五步：编译，执行make命令，安装gcc

- 第六步：编译完成后，安装，执行make install命令！

![image-20200710193506382](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710193506.png)

![image-20200710193628111](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710193628.png)

### 1.3 启动

#### 1.3.1 启动服务端

1. 设置后台启动，修改/opt/module/redis-3.2.5/redis.conf文件

```shell
[atguigu@hadoop105 redis-3.2.5]$ vim redis.conf 

修改：daemonize no 给为 daemonize yes
```

<img src="https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710194125.png" alt="image-20200710194005257" style="zoom:50%;" />

2. 启动redis服务端

```shell
[root@hadoop105 redis-3.2.5]# redis-server redis.conf
```

3. 查询服务端是否开启

```shell
[root@hadoop105 redis-3.2.5]# netstat -nap |grep 6379
```

![image-20200710194829616](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710194829.png)

4. 关闭服务端

```
在redis的客户端中，使用shurdown，关闭服务器。
```

![image-20200710195024245](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710195048.png)

5. redis服务端的端口号为：6379

<img src="https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710195237.png" alt="image-20200710195221759" style="zoom: 67%;" />

#### 1.3.2 客户端登入

```sql
-- 1. 客户端说明：
    1. 由于redis是kv的数据存储结构，那么k是不能重复的，所以redis默认有16个数据库，以便可以存储相同的k
    2. 数据库的dbid：0-15，默认情况下是使用0号数据库。
```

```sql
--2. 连接客户端：
   '方式1'：redis-cli :此时默认是使用本机的hostlocal用户登入redis。
   '方式2': redis-cli -h hadoop102 -p 6379    --hadoop105也可以使用具体的ip地址
   
   说明：默认情况下仅支持通过localhost的方式登录，如果想要其他的网卡也能登入，
         需要修改如下/opt/module/redis-3.2.5/redis.conf文件：
         将blind ：127.0.0.1 改为 0.0.0.0 ，则支持所有的网卡连接。
 -- 3. 退出客户端：
     'exit'
 -- 4. 进入客户端以后测试是否连通：
     'ping'
```

![image-20200710200054907](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200710200054.png)

## 二、redis基本操作

### 2.1 基本操作

```sql
-- 1. 切换数据库
     1. select + dbid
       默认是0号数据库
-- 2. 清空数据库
     1. flushdb：清空当前数据库
     2. flushall：清空所有数据库
-- 3. 查看当前数据的数据个数
     1. dbsize
```

### 2.2 key操作

> ==redis所有的数据都是存储在内存中==

```sql
-- 1. 获取当前数据库key
     1. keys *      --> 获取所有的key
     2. keys k*     --> 获取以k开头的所有的key
-- 2. 判断key是否存在
     1. exists key  --> 如果存在返回1，不存在返回0
-- 3. 判断key的value的数据类型
     1. type  key 
-- 4. 删除key
     1. del key
-- 5. 获取指定key的value
     1. get key     --> 只使用于value的类型为string
-- 6. 获取一个随机的key
     1. randomkey
-- 7. 设置kv键的过期时间
     1. expire key time   --> 单位为s，过了指定的时间以后，该数据就过期了，不再存在
     2. ttl key     --> 查看该key还能存活的时长：
                        如果返回-1，说明永久有效
                        如果返回-2，说明已过期
                        如果返回其他正数，说明还能存活的时长。
 -- 8. 重命名key
       1. rename oldkey newkey ：直接改，如果newkey存在，直接覆盖。
       2. renamenx oldkey newkey : 如果newkey已经存在，则不会做任何操作
```

### 2.3 value的五大常用类型

| key                | value                    |
| ------------------ | ------------------------ |
| String             | 字符串                   |
| list               | 可以重复的集合           |
| set                | 不可以重复的集合         |
| hash               | 类似于Map<String,String> |
| zset（sorted set） | 带分数的set              |

#### 2.3.1 String

```sql
-- 1. 说明
   1. String类型是Redis中最基本的类型，它是key对应的一个单一值
   2. redis的string可以包含任何数据，比如jpg图片或者序列化的对象
   3. Redis中一个字符串值的最大容量是512M
```

- 具体的操作

```sql
-- 1. 添加数据
     1. set key value
     2. setnx key value    --> 如果key存在，则添加失败
     3. setex key time value  --> 添加数据的同时设置过期时间，单位是s
     4. setrange key startindex endindex --> 在原有的value进行修改
-- 2. 批量添加数据
     1. mset  key1 value1 key2 value2 ...
     2. msetnx key1 value1 key2 value2 ...   --> 只要有一个key之前已经存在，则添加失败
-- 3. 获取数据
     1. get key 
     2. mget key1 key2 key3 ..
-- 4. 在原有的value基础上进行追加数据
     1. append key value 
-- 5. 获取value的字符长度
     1. strlen key
-- 6. 获取vlaue指定字符索引的数据
     1. getrang key startindex endinx  --> 左闭右闭
-- 7. 获取后重置value
     1. getset key newvalue  --> 先获取指定key的value后，再将原来的value使用newvalue进行替换
-- 8. ++ 与 -- 
     1. incr key  --> key对应的value自增1，要求value具体的值为数值类型
     2. decr key  --> 自减1
-- 9. 按步长增、减
     1. incrby key num  --> key对应的value + num
     2. decrby key num  --> key对应的value - num
```

#### 2.3.2 list

```sql
-- 1. list结构说明
       1. '在Java中'：list 一般是单向链表，如常见的Arraylist，只能从一侧插入。
       2. '在Redis中'：list是双向链表。可以从两侧插入
-- 2. 常见操作
       1. 遍历： 从左往右取值
       2. 删除： 弹栈 pop  ：lpop rpop
       3. 添加： 压栈 push ：lpush、rpush
   -- 说明：
      l：从左边
      r：从右边
```

- 具体的操作

```sql
-- 1. 添加数据
        1. lpush key value   --> 从左边加数据
        2. lpush key value1 value2 ...  
           --> 从左边开始，添加多个数据，但是先添加value1，再添加value2，所以value2在value1的左边。
        3. rpush key value   --> 从右边加数据
-- 2. 遍历数据
       '说明'：在redis中list中的数据有序的，所以有索引，但是索引角标的方式有两种
       '案例'： list中的数据： "b" "c" "b"
                正数角标：     0   1    2
                负数角标：     -3  -2  -1
        如上正数角标和负数角标的索引值均可以获取到list集合中数据。
       1. lindex key 角标 --> 从左边开始数，获取指定角标的数据，如果是负数，那么获取倒数第几个数据
          如：lindex list 2    --> 从左往右看，获取第三个数据
              lindex list -1   --> 从左往右看，获取倒数第一个数据
       2. lrange key 0 -1 --> 获取所有的数据，左闭右闭
-- 3. 删除数据
       1. lpop key --> 从左往右看，删除第一个数据
-- 4. 获取key对应value的list集合的长度
      1. llen key
-- 5. 在某个value的前面或者后面插入数据
      1. linsert key before/after value newvalue   --> 如果value值不存在，则插入数据失败
-- 6. 删除多个value
      1. lrem key num value --> 删除num个value
-- 7. 删除首尾的数据
      1. ltrim key startnum endnum  --> 保留[startnum ,endnum ]数据
-- 8. 删除list1最后一个数据插入到list2集合中的第一个位置
      1. rpoplpush list1 list2
```

#### 2.3.3 set

```sql
-- 1. set：无序不可重复
-- 2. 基本操作如下：
```

```sql
-- 1. 添加数据
   1. sadd key [member...]
-- 2. 获取set中所有数据
   1. SMEMBERS key  --> 获取当前key中的所有value
-- 3. 删除数据
   1. spop key       --> 删除一个数据
   2. spop key count --> 删除多个
-- 4. 判断数据是否存在
   1. SISMEMBER key member
-- 5. 获取随机的值
   1. srandmember key count --> 随机获取key中的count个value
-- 6. 求两个set的交集
   1. sinter key1 key2 ...  --> 求多个key的value的交集
   2. sinterstore destination key1 key2 ... --> 求多个key的value的交集并存放到destination中的value
-- 7. 求两个set的差集
   1. sdiff key1 key2  --> 求两个key的value的差集
   2. sdiffstore destination key1 key2 ... --> 求多个key的value的差集并存放到destination中的value，注意前后顺序
-- 8. 求两个set的并集
   1. sunion key1 key2  --> 求两个key的value的交集
   2. sunionstore destination key1 key2 ... --> 求多个key的value的交集并存放到destination中的value
-- 9. 查看set中元素的个数
   1. scard key 
```

#### 2.3.4 hash

```sql
-- 1. hash数据的结构为：map<string,string>
   key <key,value>,<key,value>  --> key <field,value> , <field,value>....
-- 2. 具体的操作：
```

```sql
-- 1. 添加数据
    1. hset key field value -->添加一个元素
    2. hmset key field value [field value ...]  -->添加多个元素
    3. hsetnx key field value --> 如果field存在，则不添加
-- 2. 获取元素
    1. hget key field  --> 获取key中的指定field的value
    2. hmget key field1 field2 ... --> 获取key中的多个field的value
    3. hgetall key --> 获取所有的hash
    4. hkeys key --> 获取所有的field
    5. hvals key --> 获取所有的value
-- 3. 删除key中field
    1. hdel key field  --> 删除key中的指定field
-- 4. 将key的field的value增加值
    1. hincrby key field number  --> 将key中的field的value增加number
-- 5. 获取hash键值对的数量
    1. hlen key 
-- 6. 判定key的field是否存在
    1. hexists key field  [field ...]
```

#### 2.3.5 zset

```sql
-- 1. zset是什么？
    1. 是一种特殊的set（sorted set），在保存value的时候，为每个value多保存了一个score信息。根据score信息，可以进行排序。
   2. 默认按照分数的从低到高进行排序。
   3. 集合的成员是唯一的，但是评分可以重复
-- 2. 结构
   key set<string,score> 
-- 3. 基本操作
```

```sql
-- 1. 添加数据
    1. ZADD key  [score member ...]
-- 2. 在分数的指定区间内返回数据，从小到大排列
    1. ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
-- 3. 在分数的指定区间内返回数据，从大到小排列
    1. ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
-- 4. 返回指定值的分数
    1. ZSCORE key member
-- 5. 统计分数区间内的元素个数
    1. ZRANGE key start stop [WITHSCORES]
-- 6. 返回集合中所有的元素的数量
    1. ZCARD key  
--  7. 统计分数区间内的元素个数
    1. ZCOUNT key min max
-- 8.  删除该集合下，指定值的元素
    1. ZREM key member
-- 9. 返回该值在集合中的排名，从0开始
    1.ZRANK key member
-- 10. 为元素的score加上增量
    1.  ZINCRBY key increment member
```

## 三 、 Redis的配置说明

### 3.1 单位说明

![image-20200712164640498](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200712164647.png)

### 3.2 include

```sql
-- 1. 说明：
    可以将公共的配置放入到一个公共的配置文件中，然后通过子配置文件引入父配置文件中的内容！将配置按照模块分开！
```

![image-20200712164718797](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200712164718.png)

### 3.3 network

```sql
-- 1. 属性 ：bind
      '含义' ：限定访问的主机地址
      '备注' ：如果没有bind，就是任意ip地址都可以访问。生产环境下，需要写自己应用服务器的ip地址。
-- 2. 属性 ：protected-mode
      '含义' ：安全防护模式
      '备注' ：如果没有指定bind指令，也没有配置密码，那么保护模式就开启，只允许本机访问。
-- 3. 属性 ：port
      '含义' ：端口号
      '备注' ：默认是6379
-- 4. 属性 ：tcp-backlog
      '含义' ：网络连接过程中，某种状态的队列的长度
      '备注' ：redis是单线程的，指定高并发时访问时排队的长度。超过后，就呈现阻塞状态。
              可以理解是一个请求到达后至到接受进程处理前的队列长度。（一般情况下是运维根据集群性能调控）高并发情况下，
              此值可以适当调高。
-- 5. 属性 ：timeout
      '含义' ：客户端连接以后长时间没有操作，那么就关闭连接，这个超时时间就是指客户端多长时间没有操作
      '备注' ：默认永不超时
-- 6. 属性 ：tcp-keepalive
      '含义' ：对客户端的心跳检测间隔时间
```

### 3.4 general

```sql
-- 1. 属性 ：daemonize
      '含义' ：是否为守护进程模式运行，也就是后台运行
      '备注' ：守护进程模式可以在后台运行
-- 2. 属性 ：pidfile
      '含义' ：进程id文件保存的路径
      '备注' ：配置PID文件路径，当redis作为守护进程运行的时候，它会把 pid 默认写到 /var/redis/run/redis_6379.pid 文件里面
-- 3. 属性 ：loglevel
      '含义' ：定义日志级别
      '备注' ：一共有4种存储级别，根据用途不用选择不同的存储级别
               级别1：debug（记录大量日志信息，适用于开发、测试阶段） 
               级别2：verbose（较多日志信息） 
               级别3：notice（适量日志信息，使用于生产环境）
               级别4：warning（仅有部分重要、关键信息才会被记录）
-- 4. 属性 ：syslog-enabled
      '含义' ：是否记录到系统日志
      '备注' ：要想把日志记录到系统日志服务中，就把它改成 yes
-- 5. 属性 ：databases
      '含义' ：设置数据库数量
      '备注' ：默认16个数据库，数据库的dbid：0-15
-- 6. 属性：logfile
      '含义' ：日志目录
      '备注' ：前台运行时打印在控制台，后台运行时默认写到黑洞里面。所以我们一般会配置一个路径
```

### 3.5 其他

```sql
-- 1. 属性 ：requirepass
      '含义' ：设置密码
-- 2. 属性 ：maxclients
      '含义' ：最大连接数
-- 3. 属性 ：maxmemory
      '含义' ：最大占用多少内存
      '备注' ：一旦占用内存超限，就开始根据缓存清理策略移除数据如果Redis无法根据移除规则来移除内存中的数据，
              或者设置了“不允许移除”，那么Redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。
-- 4. 属性 ：maxmemory-policy noeviction
      '含义' ：缓存清理策略
      根据使用redis的用途不同，选择不同的清除策略。
      作为数据库时：noeviction，永远不要删除
      作为缓存时：volatile-lru或者allkeys-lru或者volatile-ttl，最好是volatile-ttl
      作为消息的中间件时：
      '备注' ：一共有6种清除，清除方式如下：
      'LRU算法：最近最少使用的策略'。
      （1）volatile-lru：使用LRU算法移除key，只对设置了过期时间的键
      （2）allkeys-lru：使用LRU算法移除key
      （3）volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
      （4）allkeys-random：移除随机的key
      （5）volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
      （6）noeviction：不进行移除。针对写操作时会返回错误信息       
-- 5. 属性 ：maxmemory-samples
      '含义' ：样本数，默认是5个
      '备注' ：样本数越小，准确率越低，但是性能越好。
             LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小。一般设置3到7的数字。

```

## 四、 持久化

```sql
-- 1. 持久化介绍
  redis的数据是基于内存，一旦出现突然断电，那么数据就会从内存中丢失，所以redis提供了两种数据持久化的方式:rdb和aof。
-- 2. 两种持久化方法的主要区别：
   1. rdb：按照保存策略每间隔一段时间将redis内存中数据做一个快照，保存到磁盘中，但是前面写完快照以后就会把之前的快照删除，只保留最新的快照数据；
   2. aof: 按照保存策略，将redis的对'数据的操作指令'保存到磁盘中，可以恢复已删除的数据。
-- 3. 两种持久化方法的优劣势待详细分解后归纳总结。
```

### 4.1 RDB

#### 4.1.1 RDB介绍

```sql
-- 1. RDB是什么？
      '持久化方式'：在指定的时间间隔内将redis内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，
      '恢复数据方式'：将快照文件直接读到内存里
-- 2. 工作机制：
      每隔一段时间，就把内存中的数据保存到硬盘上的指定文件中
-- 3. 默认是开启状态
-- 4. 实现方式：
       第一步：Redis会单独创建（fork）一个子进程来进行持久化，
       第二步：先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。
       '说明'：
       1. 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的恢复
       2. 对于数据恢复的完整性不是非常敏感
-- 5. 缺点
      最后一次持久化后的数据可能丢失
```

#### 4.1.2 RDB的持久化触发条件

```sql
-- 1. 触发条件1：自动保存策略，如下三种情况任何一种满足即进行持久化操作
    save <seconds> <changes> :在多少秒内数据发生改变以后就会自动保存
    '情况1'：save 900 1     :900 秒内如果至少有 1 个 key 的值变化，则保存
    '情况2'：save 300 10    :300 秒内如果至少有 10 个 key 的值变化，则保存
    '情况3'：save 60 10000  :60 秒内如果至少有 10000 个 key 的值变化，则保存
-- 2. 其他触发条件：
     1. 执行save，或者bgsave命令！执行时，是阻塞状态。
	 2. 执行flushall命令，也会产生dump.rdb，只是将内存中的数据清空并进行持久化
	   执行flushdb，只会清空当前数据库的数据，不会进行持久化
	 3. 当执行shutdown命令时，也会主动地备份数据
-- 3. 持久化的文件也可以实现压缩。
```

#### 4.1.3 参数配置

```sql
-- 1. 属性 ：save
      '含义' ：保存策略
-- 2. 属性 ：dbfilename
      '含义' ：RDB快照文件名
-- 3. 属性 ：dir
      '含义' ：RDB快照保存的目录
      '备注' ：必须是一个目录，不能是文件名。最好改为固定目录。默认为./代表执行redis-server命令时的当前目录！
-- 4. 属性 ：stop-writes-on-bgsave-error
      '含义' : 是否在备份出错时，继续接受写操作
      '备注' ：如果用户开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求 
-- 5. 属性 ：rdbcompression
      '含义' ：对于存储到磁盘中的快照，可以设置是否进行压缩存储。
      '备注' ：如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，
               但是存储在磁盘上的快照会比较大。
               读取压缩文件，需要消耗内存
-- 5. 属性 ：rdbchecksum
      '含义' ：是否进行数据校验
      '备注' ：在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗， 
               如果希望获取到最大的性能提升，可以关闭此功能。
```

#### 4.1.4 RDB的优缺点

```sql
-- 1. 优点
   1. 持久化的过程会单独创建一个子线程来完成，不影响主线程的io
   2. RDB是一个非常紧凑的文件，很适合直接发送到其他的远端数据中心来进行备份
   3. 对比aof持久化过程，数据恢复更快一些。

-- 2. 缺点
   1. 每生成一个新的rdb文件时，旧的rdb文件就会删除
   2. 最后一次的持久化过程，可能会丢失数据
   3. RDB每次持久化时，需要fork一个新的子线程来完成持久化的过程，消耗一定的性能

```

### 4.2 AOF

#### 4.2.1 AOF简介

```sql
-- 1. 持久化的方式：
     AOF是以日志的形式来记录每个写操作，将每一次对数据进行修改，都把新建、修改数据的命令保存到指定文件中。
-- 2. 数据恢复的方式：
    Redis重新启动时读取这个文件，重新执行新建、修改数据的命令恢复数据。
-- 3. 默认不开启，需要手动开启
-- 4. AOF文件的保存路径，同RDB的路径一致。
-- 5. AOF在保存命令的时候，只会保存对数据有修改的命令，也就是写操作！
-- 6. 当RDB和AOF存的不一致的情况下，按照AOF来恢复。因为AOF是对RDB的补充。备份周期更短，也就更可靠。
```

#### 4.2.2 AOF保存数据的策略

```sql
-- 1. AOF保存策略
1. appendfsync always：每次产生一条新的修改数据的命令都执行保存操作；效率低，但是安全！
2. appendfsync everysec：每秒执行一次保存操作。如果在未保存当前秒内操作时发生了断电，仍然会导致一部分数据丢失（即1秒钟的数据）。
3. appendfsync no：从不保存，将数据交给操作系统来处理。更快，也更不安全的选择。

-- 2. 推荐（并且也是默认）的措施为每秒 fsync 一次， 这种 fsync 策略可以兼顾速度和安全性。
```

#### 4.2.3 配置参数

```sql
-- 1. 属性 ：appendonly
      '含义' ：是否开启AOF功能
      '备注' ：默认是关闭的 
-- 2. 属性 ：appendfilename
      '含义' ：AOF文件名称
-- 3. 属性 ：appendfsync
      '含义' ：AOF保存策略
      '备注' ：官方建议everysec
-- 4. 属性 ：no-appendfsync-on-rewrite
      '含义' 在重写时，是否执行保存策略
      '备注' ：aof重写功能如果开启，那么会将aof的数据加载到内存中，进行aof文件中的数据进行优化，可以节省aof文件的大小。
              此参数为：执行重写，是否支持持久化
-- 5. 属性 ：auto-aof-rewrite-percentage
      '含义' ：重写的触发条件1
      '备注' ：当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写
-- 6. 属性 ：auto-aof-rewrite-min-size
      '含义' ：设置允许重写的最小aof文件大小
      '备注' ：避免了达到约定百分比但尺寸仍然很小的情况还要重写
-- 7. 属性 ：aof-load-truncated
      '含义' ：截断设置
      '备注' ：如果AOF -load-truncated设置为yes，则加载一个截断的AOF文件，并且Redis服务器开始发送日志通知用户事件。
			 如果选项被设置为no，服务器将终止并出现错误拒绝开始。
			 当选项设置为no时，用户需要在重启之前使用"redis-check-aof"实用程序修复AOF文件
```

#### 4.2.4  AOF文件的修复

```sql
如果AOF文件中出现了残余命令，会导致服务器无法重启。此时需要借助redis-check-aof工具来修复！

命令： redis-check-aof  --fix 文件
```

#### 4.2.5 AOF的优缺点

```sql
-- 1. 优点
    1. 备份机制更稳健，丢失数据概率更低
    2. aof持久化文件是可读的日志文本，通过操作AOF稳健，可以处理误操作

-- 2. 缺点
    1. 比起RDB占用更多的磁盘空间
    2. 恢复备份速度要慢
    3. 每次读写都同步的话，有一定的性能压力
```

### 4.3 官方持久化建议

```sql
-- 官方推荐两个都用；如果对数据不敏感，可以选单独用RDB；不建议单独用AOF，因为可能出现Bug;如果只是做纯内存缓存，可以都不用
```

## 五、事务

### 5.1 redis的事务介绍

```sql
1. Redis中事务，不同于传统的关系型数据库中的事务,acid一个都没有
2. Redis中的事务指的是一个单独的隔离操作。
3. Redis的事务中的所有命令都会序列化、按顺序地执行且不会被其他客户端发送来的命令请求所打断。
4. Redis事务的主要作用是串联多个命令防止别的命令插队
```

### 5.2 redis常用的命令

```sql
-- 1. MULTI ： 标记一个事务块的开始
-- 2. EXEC ： 执行事务中所有在排队等待的指令并将链接状态恢复到正常 
              当使用WATCH 时，只有当被监视的键没有被修改，且允许检查设定机制时，EXEC会被执行
-- 3. DISCARD ：刷新一个事务中所有在排队等待的指令，并且将连接状态恢复到正常。
                如果已使用WATCH，DISCARD将释放所有被WATCH的key  
-- 4. WATCH：标记所有指定的key 被监视起来，在事务中有条件的执行（乐观锁）
```

### 5.3 redis组队失败

```sql
-- 1. 情况1：自作自受：类似java中的运行时错误，只有运行错误的操作指令不会执行，其他的指令会正常执行

-- 2. 情况2：殃及池鱼：事务中语法出现错误，类似编译时就报错了，那么事务块中所有的指令都不会执行

-- 3. 官方说明:为什么情况1不支持rockback，回滚呢？
      因为在redis就是基于内存的操作，要求处理的数据快，所以redis内部可以保持简单且高速。
```

### 5.4 Redis中的锁

```sql
-- Redis中的锁策略
  1. Redis采用了乐观锁策略（通过watch操作）。乐观锁支持读操作，适用于多读少写的情况！
  2. 在事务中，可以通过watch命令来加锁；使用 UNWATCH可以取消加锁；
  3. 如果在事务之前，执行了WATCH（加锁），那么执行EXEC 命令或 DISCARD 命令后，锁对自动释放，即不需要再执行 UNWATCH 了
```

## 六、主从复制

### 6.1 主从简介

```sql
-- 1. 什么是主从复制
    配置多台Redis服务器，以主机和备机的身份分开。主机数据更新后，根据配置和策略，自动同步到备机的master/salver机制。
-- 2. 主从用途
     Master以写为主，
     Slave以读为主，
     二者之间自动同步数据。
-- 3. 主从目的
    1. 读写分离提高Redis性能；
    2. 避免单点故障，容灾快速恢复
-- 4. 实现原理
   1. 当从机第一次上线时：从机给主机发送sync指令，主机立刻进行存盘操作，发送RDB文件给到从机，从机收到RDB文件以后，就会将数据
      加载到内存汇总。
   2. 后续：每次主机的写操作命令，都会立刻发送给从机，从机执行相同的命令来保持和主机的数据一致。
-- 5. 注意
    1. 主机收到从机的sync同步的指令时，即使在配置文件中禁止使用RDB持久化时，也会生产RDB文件
    2. 但是如果主机所在的服务器磁盘的IO性能比较差时，那么这个复制过程就会出现瓶颈。
    3. 第二点的问题，redis在2.8.18版本开始实现了无磁盘复制功能。不过该功能还是处于试验阶段
       '实现方式'：设置参数：repl-diskless-sync yes
       '实现过程'：redis的主机进行复制初始化，不会将内存的数据进行持久化，而是直接通过网络的方式直接发送给从机，避免了io性
       能差的问题.
```

### 6.2 主从准备

```sql
-- 1. 除非是不同的主机配置不同的Redis服务，否则在一台机器上面跑多个Redis服务，需要配置多个Redis配置文件
-- 步骤1：准备多个Redis配置文件，每个配置文件
   0. 在家目录在创建mkdir my_redis目录，在这个目录下创建如下4个配置文件
   1. 准备一个base.conf文件
   2. 另外准备3个配置文件：6379.conf/6380.conf/6381.conf
   3. 如上4个配置文件的内容为：
```

- 6379.conf

```sql
include /home/atguigu/my_redis/base.conf
dbfilename dump_6379.rdb
pidfile /home/atguigu/my_redis/6379.pid
logfile "/home/atguigu/my_redis/6379.log"
port 6379
```

- 6380.conf

```sql
include /home/atguigu/my_redis/base.conf
dbfilename dump_6380.rdb
pidfile /home/atguigu/my_redis/6380.pid
logfile "/home/atguigu/my_redis/6380.log"
port 6380
```

- 6381.conf

```sql
include /home/atguigu/my_redis/base.conf
dbfilename dump_6381.rdb
pidfile /home/atguigu/my_redis/6381.pid
logfile "/home/atguigu/my_redis/6381.log"
port 6381
```

```sql
-- 如上3个配置文件的说明：
include    -- 可以将公共的配置放入到一个公共的配置文件中，然后通过子配置文件引入父配置文件中的内容！将配置按照模块分开！
dbfilename -- 持久化文件名字
pidfile    -- pid文件路径
logfile    -- log文件路径
port 6381  -- redis端口
```

- base.conf

```sql
-- 是从redis-3.2.5目录中拷贝过来的，主要参数有：
   1. bind 0.0.0.0
   2. protected-mode no
   3. port 6379
   4. tcp-backlog 511
   5. timeout 0
   6. tcp-keepalive 300
   7. daemonize yes
   8. supervised no
   9. pidfile /var/run/redis_6379.pid
   10. loglevel notice
   11. logfile "/opt/module/redis-3.2.5/redis.log"
   12. databases 16
   13. save 900 1
       save 300 10
       save 60 10000
   14. stop-writes-on-bgsave-error no
   15. rdbcompression yes
   16. rdbchecksum yes
   17. dbfilename dump_7379.rdb
   18. dir /home/atguigu/my_redis --> rdb文件路径
   19. slave-read-only yes   
   20. repl-diskless-sync no      --> 同步时是否直接通过网络进行发送
   21. appendonly no   --> 不开启aof持久化
   22. appendfsync everysec --> aof持久化策略
   22. no-appendfsync-on-rewrite no --> 重写的过程依然执行持久化
   23. auto-aof-rewrite-percentage 100  -->重写触发条件，重写文件是上一个重写文件的百分比
   24. auto-aof-rewrite-min-size 64mb  --> aof重写触发条件：重写文件的最小大小
   25. aof-load-truncated yes   --> 是否加载截断的文件
```

```sql
-- 步骤2：分别启动不同配置文件的redis-server服务端
```

![image-20200713002901981](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200713002902.png)

![image-20200713001313427](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200713001320.png)

```sql
-- 步骤3： 分别进入不同redis-server的客户端并查看各自的信息。3个客户端的情况是一致的。
          查看主从的副本关系：info replication
```

![image-20200713002037712](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200713002037.png)

> 主从关系的配置：配从不配主。

### 6.3 主从建立

#### 6.3.1 建立临时的主从关系

```sql
-- 1. 临时建立
    '原则'：配从不配主。
    '配置'：在从服务器上执行'SLAVEOF ip:port'命令；
    '查看'：执行'info replication'命令；
-- 2. slave:只支持读的操作，不支持写的操作，同时主机的数据，在从机上能够直接获取
     hadoop105:6379> set k1 2
	(error) READONLY You can't write against a read only slave.
```

![image-20200713003444616](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200713003444.png)

![image-20200713003653998](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200713003654.png)

#### 6.3.2 永久建立

```sql
-- 1. 直接在配置文件指明当前的redis属于哪个的slave。
    在从机的配置文件中，编写slaveof属性配置！
    如在6380.conf配置中写上： slaveof hadoop105 6379
```

#### 6.3.3 恢复身份

```
  执行命令slaveof no one恢复自由身！
```

### 6.4 常见问题

```sql
-- 1. 从机是从头开始复制主机的信息，还是只复制切入以后的信息？
   1. 从机第一次启动，从头开始复制主机信息
   2. 后续，主机有一条数据发生修改，就发送给从机
-- 2. 从机是否可以写？
   1. 默认情况不能
   2. 可以通过修改配置文件，设置slave-read-only no.但是从机写入的数据是不能同步到主机，因此没用设置的必要！
-- 3. 主机shutdown后，从机是上位还是原地待命？
	答： 原地待命
-- 4. 主机又回来了后，主机新增记录，从机还能否顺利复制？
    答：可以
-- 5. 从机宕机后，重启，宕机期间主机的新增记录，从机是否会顺利复制？
	答：可以
-- 6. 其中一台从机down后重启，能否重认旧主？ 
	答：不一定，如果是临时主从关系，不能重认旧主，如果是在配置文件中，指定了master，那么可以重认旧主
-- 7. 如果两台从机都从主机同步数据，此时主机的IO压力会增大，如何解决？ 
	答：按照主---从（主）---从模式配置！
```

### 6.5 哨兵模式

## 七、集群模式