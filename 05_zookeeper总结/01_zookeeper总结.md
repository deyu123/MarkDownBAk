#                                               zookeeper总结

******************

总结：     时间：2020.05.02

## 一、zookeeper入门

### 1.1 概述

> ![image-20201109203507352](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109203507352.png)

```mysql
-- 理解
1) Apache一个分布式项目；
2）是一个基于观察者模式设计的分布服务管理框架，负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦数据发生变化，则zookeeper通知观察者。

-- zookeeper = 文件系统 + 通知机制。

```

### 1.2 zookeeper特点

> ![image-20201109203648043](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109203648043.png)

```mysql
-- 特点
1)  每一节点都有一个不重复的myid标识zookeeper集群；
2） 一个领导者（leader） 和 n 个追随者 （follower）组成的集群；
3） 集群只要有半数以上的节点存活就能对外提供服务，即使是在leader失败的情况下；-- 半数机制
4） 根据半数机制，可知一般搭建奇数台的机器是有优势的；
	假设是4台，半数为2，故障一台，还有3台，3 > 2 ,所以继续提供服务，再故障一台，剩下2台， 2 > 2 ,错误，此时，不再对外提供服务；
	假设是 3台，半数为 1.5 ，故障1台，还有2台， 2 > 1.5 , 所以继续提供服务，再故障一台，剩下1台， 1 > 1.5 ,错误，此时不再对外提供服务。
	综上，4台机器，允许故障1台，3台机器，也允许故障一台，所以奇数台机器有优势。
5） 全局数据一致：所有节点的数据是保持一致的，所以客户端无论连接到哪台机器，获取的数据都是一样的； -- 基于zab机制；
6） 更新请求按照顺序执行；
7） 数据更新的原子性：要么成功，要么失败；
8） 实时性： 在一定时间范围内，client能够获得最新的数据。

```

### 1.3 zookeeper数据结构

> ![image-20201109203725535](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109203725535.png)

```mysql
1） 与linux文件结构相似，也是一个根目录。整体呈一颗树的形状；
2） 每一个节点称作一个znode，每一个znode可以存储1M的数据；
3） 每个znode通过其路径进行唯一标识。
```

### 1.4 应用场景

> ![image-20201109203802932](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109203802932.png)

> ![image-20201109203905683](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109203905683.png)

> ![image-20201109203925492](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109203925492.png)

> ![image-20201109203948438](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109203948438.png)

> ![image-20201109204005044](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109204005044.png)

```mysql
1） 统一命名服务；
2） 统一配置管理；
3） 服务器节点动态上下线；
4） 软负载均衡。
```

### 1.5  配置参数解读

```mysql
1）tickTime =2000：通信心跳数，Zookeeper服务器与客户端心跳时间，单位毫秒
Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。
它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)
2）initLimit =10：LF初始通信时限
集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。
3）syncLimit =5：LF同步通信时限
集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。
4）dataDir：数据文件目录+数据持久化路径
主要用于保存Zookeeper中的数据。
5）clientPort =2181：客户端连接端口
监听客户端连接的端口。
```

1.6 zookeeper的四字命令

```mysql
需要在Zookeeper的配置文件/opt/module/zookeeper-3.5.7/conf/zoo.cfg中加入如下配置:
4lw.commands.whitelist=*
-- 语法：
连接方式： nc  机器名 端口号 （如果nc功能没有，则使用yum进行安装）
-- 示例：
nc hadoop102 2181
```

| ruok | 测试服务是否长度处于正确状态，如果确实如此，那么服务返回 imok ,否则不做任何响应。 |
| ---- | ------------------------------------------------------------ |
| conf | 3.3.0版本引入的，打印出服务相关配置的详细信息                |
| cons | 列出所有连接到这台服务器的客户端全部会话详细信息。包括 接收/发送的包数量，会话id，操作延迟、最后的操作执行等等信息 |
| crst | 重置所有连接的连接和会话统计信息                             |
| dump | 列出那些比较重要的会话和临时节点。这个命令只能在leader节点上有用 |
| envi | 打印出服务环境的详细信息                                     |

## 二、zookeeper内部原理

### 2.1  节点类型

> ![image-20201109204135846](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201109204135846.png)

```mysql
分为持久性和短暂性节点。
1） 持久性：客户端与服务器断开连接以后，创建的节点不删除；
	-- 分为带序号的持久性节点和不带序号的持久性节点。
2） 短暂性：客户端与服务器断开连接以后，创建的节点删除；
	-- 分为带序号的短暂性节点和不带序号的短暂性节点。
	
-- 如何理解带序号的呢？
1） 在zk分布系统中，顺序号用于为所有事物进行全局排序，这样客户端根据顺序号推测事件的顺序。
2） 顺序号是指当前节点下的节点数量，不可重复使用。如之前已经创建了一个节点，但是现在将其进行删除，再创建一个节点，顺序号是往后进行累加。

-- 说明
1） 短暂节点下不能创建子节点；
2） 一个节点包含该节点的具体存储的内容和子节点的元数据信息。

```

### 2.2  Stat结构体

```mysql
1）czxid-创建节点的事务zxid
每次修改ZooKeeper状态都会收到一个zxid形式的时间戳，也就是ZooKeeper事务ID。
事务ID是ZooKeeper中所有修改总的次序。每个修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1在zxid2之前发生。
2）ctime 被创建的毫秒数(从1970年开始)
3）mzxid 最后更新的事务zxid
4）mtime 最后修改的毫秒数(从1970年开始)
5）pZxid-znode最后更新的子节点zxid
6）cversion 子节点变化号，znode子节点修改次数
7）dataversion 数据变化号
8）aclVersion 访问控制列表的变化号
9）ephemeralOwner- 如果是临时节点，这个是znode拥有者的session id。如果不是临时节点则是0。
10）dataLength   数据长度
11）numChildren  子节点数量
```

### 2.3  监听器原理（重点）

```mysql
 监听原理详解：
 1） 首先有一个main（）线程；
 2） 在main线程中创建zookeeper客户端，这时就会有两个线程，一个负责网络通信（connet），一个负责监听（listener）；
 3） 通过connect线程将注册的监听事件发送给zookeeper；
 4） 在zookeeper的注册监听器列表中将注册的监听事件添加到列表中；
 5） zookeeper监听到有数据或者是路径发生变化时，就会将这个消息发送到listener线程；
 6） listener线程内部调用process方法（）；
```

### 2.4  选举机制（重点）

```mysql
总结：选举机制由节点启动的顺序、myid、数据的zxid、服务器的数量有关。
大致顺序为：
1）zxid大的当选（99%情况下都是相等的）；
2）根据节点启动的顺序，比较myid，在未到达半数的服务器数量以前，所有节点的票都将投给myid大的服务器，一旦到达了半数以上的服务器被启动（此时可以对外提供服务）时，myid最大的节点当选leader，其余的服务器为follower。
```

### 2.5  写数据流程

```mysql
1） Client 向zookeeper申请写数据，发送一个写请求；
2） 如果这个服务器不是leader，则该服务器将接收到的请求转发给leader；
3） leader将这个写的请求广播给所有的follower服务器，所有的服务器将写的事件写入队列中，并向leader发送准备就绪的成功消息；
4） 当leader收到了半数以上的服务器返回了成功的消息以后，则说明该写的操作可以执行，则leader向所有的follower发送提交消息，所有的服务器收到信息以后则执行队列中的写操作；
5） 对应的服务器完成写操作以后会通知client，数据写入成功。
```

## 三、zookeeper实战部署

### 3.1 客户端命令行操作

```mysql
1）help : 显示所有的操作指令；
2）ls path : 使用ls命令来查看当前znode的子节点
	-w ：监听子节点变化（只能监听一次）
	
3） 查看当前目录下的详细信息：
	ls -s path /  ls2 path / stat path 
4)  create 创建节点 （默认创建的是：持久性无顺序号的节点）
	 -s 含顺序号
	 -e 临时的
5） get path : 获得节点的值
     -w  监听节点内容变化
6） set path ：设置节点具体的值
8） delete : 删除节点
9） deleteall：递归删除节点。
```

### 3.2 API应用

1. 创建Maevn Modle
2. 添加pom文件

```java
<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.8.2</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.5.7</version>
		</dependency>
</dependencies>
```

3. 需要在项目的src/main/resources目录下，新建一个文件，命名为“log4j.properties”，在文件中填入

```java
log4j.rootLogger=INFO, stdout  
log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n  
log4j.appender.logfile=org.apache.log4j.FileAppender  
log4j.appender.logfile.File=target/spring.log  
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout  
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n  
```

4. 创建zookeeper客户端

```java
	private static String connectString ="hadoop102:2181,hadoop103:2181,hadoop104:2181";
	private static int sessionTimeout = 2000;
	private ZooKeeper zkClient = null;

	@Before
	public void init() throws Exception {

		zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {

			@Override
			public void process(WatchedEvent event) {

				// 收到事件通知后的回调函数（用户的业务逻辑）
				System.out.println(event.getType() + "--" + event.getPath());

				// 再次启动监听
				try {
					zkClient.getChildren("/", true);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}
}
```

5. 创建子节点

```java
// 创建子节点
@Test
public void create() throws Exception {

		// 参数1：要创建的节点的路径； 参数2：节点数据 ； 参数3：节点权限 ；参数4：节点的类型
	String nodeCreated = zkClient.create("/atguigu", "jinlian".getBytes(),Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
}
```

6. 获取子节点并监听节点变化

```java
// 获取子节点
@Test
public void getChildren() throws Exception {

		List<String> children = zkClient.getChildren("/", true);

		for (String child : children) {
			System.out.println(child);
		}

		// 延时阻塞
		Thread.sleep(Long.MAX_VALUE);
}
```

7. 判断zonde是否存在

```java
// 判断znode是否存在
@Test
public void exist() throws Exception {

	Stat stat = zkClient.exists("/eclipse", false);

	System.out.println(stat == null ? "not exist" : "exist");
}
```

### 3.3 监听服务器节点动态上下线案例

服务器端向Zookeeper注册代码

```java
package com.atguigu.zkcase;
package com.atguigu.zookeeper;

import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.io.IOException;

/**
 *  当服务器上线后， 将当前服务器对应的信息写到zk中
 */
public class Server {

    private String connectionString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
    private int sessionTimeOut = 10000;
    private ZooKeeper zkClient = null ;
    private String  parentNode = "/servers";
    public static void main(String[] args) throws Exception {
        Server server = new Server ();
        //1. 初始化zk客户端对象
        server.init();
        //2. 判断zk中存储服务器信息的Znode是否存在
        server.parentNodeExists();
        //3. 将服务器的信息写入到zk中
        server.writeServer(args);
        //4. 保持线程不结束
        Thread.sleep(Long.MAX_VALUE);
    }

    /**
     * args中包含两个数据:
     *  1. server的名字
     *  2. server的信息
     * @param args
     */
    private void writeServer(String [] args) throws KeeperException, InterruptedException {
        String s =
                zkClient.create(parentNode + "/" + args[0], args[1].getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        System.out.println("*********** "+ s +"is on line  ************");
    }
 //2. 判断zk中存储服务器信息的Znode是否存在
    private void parentNodeExists() throws KeeperException, InterruptedException {
        Stat stat = zkClient.exists(parentNode, false);
        if(stat == null){
            //创建节点
            zkClient.create(parentNode,"servers".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
        }
    }
	//1. 初始化zk客户端对象
    private void init() throws IOException {
        zkClient = new ZooKeeper(connectionString, sessionTimeOut, new Watcher() {
            @Override
            public void process(WatchedEvent event) {

            }
        });
    }
}


```

客户端代码

```java
package com.atguigu.zkcase;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

public class DistributeClient {

	private static String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
	private static int sessionTimeout = 2000;
	private ZooKeeper zk = null;
	private String parentNode = "/servers";

	// 创建到zk的客户端连接
	public void getConnect() throws IOException {
		zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {

			@Override
			public void process(WatchedEvent event) {

				// 再次启动监听
				try {
					getServerList();
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}

	// 获取服务器列表信息
	public void getServerList() throws Exception {
		
		// 1获取服务器子节点信息，并且对父节点进行监听
		List<String> children = zk.getChildren(parentNode, true);

        // 2存储服务器信息列表
		ArrayList<String> servers = new ArrayList<>();
		
        // 3遍历所有节点，获取节点中的主机名称信息
		for (String child : children) {
			byte[] data = zk.getData(parentNode + "/" + child, false, null);

			servers.add(new String(data));
		}

        // 4打印服务器列表信息
		System.out.println(servers);
	}

	// 业务功能
	public void business() throws Exception{

		System.out.println("client is working ...");
Thread.sleep(Long.MAX_VALUE);
	}

	public static void main(String[] args) throws Exception {

		// 1获取zk连接
		DistributeClient client = new DistributeClient();
		client.getConnect();

		// 2获取servers的子节点信息，从中获取服务器信息列表
		client.getServerList();

		// 3业务进程启动
		client.business();
	}
}
```

