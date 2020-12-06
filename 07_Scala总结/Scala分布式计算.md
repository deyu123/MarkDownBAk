# 分布式计算应用

***

## 1.需求

```sql
-- 一个发送端向多个接收端发送数据，接收端接收数据以后并进行处理，最后将数据返回给发送端，发送端将计算结果收回并打印在控制台。
```

## 2.需求分析

### 2.1 搭建模型

```sql
-- 一个发送端连接多个接收端，然后分别进行发送数据，此时存在的问题：
-- 当服务器接收到数据以后，计算需要很长一段时间时，如果客户端一直等着服务器返回数据，这是不合理的。
```

![image-20200601215250678](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200601215250.png)

### 2.2 优化1

```sql
 -- 客户端发送数据给服务端以后，断开连接，等服务器计算数据完成以后，服务器去连接客户端，将计算完成以后的结果返回
```

![image-20200601215841564](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200601215841.png)

### 2.3 优化2

```
因为分布式计算的台数是根据实际的数据量来定的，数据量大，那我们就多一些服务器来进行计算，如果数量少一些，那么服务器就少一些，所以创建一个资源管理的中心，客户端需要多少资源就告诉资源管理中心，然后资源管理中心就创建多少个服务器来进行运算。
```

![img](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200601220829.png)



## 3.需要考虑的问题

### 3.1 数据如何进行切分

```sql
-- 暂时不传数据，只传任务和数据处理的逻辑
```

### 3.2 客户端如何告诉资源管理中创建服务器的台数

```sql
--客户端和资源管理中心建立网络连接，客户端将自己的端口号、地址和需要的服务器台数发送给资源管理中心
```

### 3.3 资源管理中心如何创建服务器

```sql
-- 资源管理中心根据客户端发来的端口号和地址就可以创建服务器，使用一个循环来创建服务器台数
```

### 3.4  服务器如何接收到来自客户端的数据

```sql
-- 在服务器类中，声明一个方法，方法中声明输入流，计算，输出流，并把结果输出等代码
```

### 3.5  客户端如何获取服务器端发来的计算结果

```sql
-- 在客户端处，创建发送数据的线程中，接收来自执行器的数据
```

### 3.6 客户端如何收集整合不同服务器端发来的数据

```sql
--因为在客户端，会收集来自不同服务器端的数据，那么相当于使用多个线程对同一个结果集进行操作，存在线程安全问题，那么需要使用一个容器来接收结果数据。
```



## 4.功能模块

### 4.1 驱动器

```sql
1. 将driver的host、port、执行器的数量信息传递给资源中心，关闭与资源中心的连接
2. 创建一个serversocket，并获取输出流
3. 创建线程，用于driver将任务发送给执行器和接收执行器返回的计算结果，在线程内：
       a、将需要输出的任务发送给执行器；
       b、关闭输出流
       c、获取输入流
       d、使用一个集合来接收执行器返回的数据
4. 创建另外一个线程，用于判定是否所有的执行器已经返回结果，并打印所有执行器的计算结果
```

### 4.2  资源中心

```sql
1.  获取driver的host、port、exerutorsNum
2.  根据exerutorsNum，创建对应数量的executor
3.  并将driver的host、port传递给executor
4.  前三步在一个线程里面执行
```

### 4.3信息

```sql
-- 将driver的host、port、执行器的数据封装成一个样例类。
```

### 4.4 执行器

```

1. 通过资源中心传入的参数，获取driver的host、port以及执行器的id
2. 创建一个方法，在方法内： 
       a、创建一个socket，并获取输入流
       b、获取driver的数据
       c、进行数据处理，并创建输出流
       d、将结果和执行器id返回给driver
```

### 4.5任务

```sql
-- 将数据处理的逻辑封装成一个普通类
```



![image-20200602015033371](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200602015033.png)

## 5.代码实现

### 5.1 驱动器

```scala
object Driver {
  def main(args: Array[String]): Unit = {

    //1. 与resourceCenter进行连接，并将驱动类的端口号、ip、和需要的服务台的台数的消息发送给资源中心
    //1.1 数据准备阶段，将发送的消息封装成一个样例类
    val executorNum = 5
    val driverPort = 9999
    val driverHost = "localhost"
    val message = Message(s"executorNum=${executorNum}&driverHost=${driverHost}&driverPort=${driverPort}")

    // 1.2  与resourceCenter连接,并将准备的数据发送给资源中心
    val socket = new Socket("localhost", 6666)
    val out = new ObjectOutputStream(socket.getOutputStream)
    out.writeObject(message)
    out.flush()
    socket.close()

    // 2. 创建一个服务器，任务传输给执行器
    val receiver = new ServerSocket(driverPort)

    // TODO 接收Executor端的数据
    // 线程安全问题

    val start = System.currentTimeMillis()
    val results: Array[Int] = Array.fill(executorNum)(-1)

    new Thread(
      new Runnable {
        override def run(): Unit = {
          // TODO 统计结果的线程
          var flg = true
          while (flg) {
            if (results.contains(-1)) {
              Thread.sleep(100)
            } else {
              // TODO 所有的线程都已经计算完毕
              val end = System.currentTimeMillis()
              println("计算完毕，结果为 " + results.sum + ",耗时:" + (end - start) + "ms")
              flg = false
              System.exit(0)
            }
          }
        }
      }
    ).start()


    while (true) {
      val executorRef: Socket = receiver.accept()
      new Thread(new Runnable {
        override def run(): Unit = {

          // 2.1 发送任务给到执行器
          val exerutorOutputStream = new ObjectOutputStream(executorRef.getOutputStream)
          val task = new Task
          task.func = _ * 2
          exerutorOutputStream.writeObject(task)
          exerutorOutputStream.flush()
          exerutorOutputStream.close()
          executorRef.shutdownInput()

          //2.2 获取执行器的返回的结果
          // TODO 获取Executor端计算结果
          val executorRefIn =
          new ObjectInputStream(executorRef.getInputStream)
          val message: Message = executorRefIn.readObject().asInstanceOf[Message]
          val datas: Array[String] = message.m.split("=|&")
          // executorId=${id}&result=$i
          // executorId
          // id
          // result
          // i
          results(datas(1).toInt-1) = datas(3).toInt
          //println("获取计算结果 = " + result)
        }
      }).start()

    }


  }

}

```

### 5.2 资源中心

```scala
object ResourceCenter {
  def main(args: Array[String]): Unit = {

    // 1. 与driver进行连接，获取driver的端口号，ip、需要的服务器的台数
    val socket = new ServerSocket(6666)
    while (true){
      val driver: Socket = socket.accept()
      new Thread(new Runnable {
        override def run(): Unit = {
          val in = new ObjectInputStream(driver.getInputStream)
          val unit: AnyRef = in.readObject()
          val message: Array[String] = unit.asInstanceOf[Message].m.split("=|&")
          //"executorNum,5,driverHost,localhost,driverPort,1234"

          println(message.mkString(","))
          val exerutorCount: Int = message(1).toInt
          val driverHost = message(3)
          val driverPort  = message(5).toInt


//           2.创建服务器
          for (id <-  1 to exerutorCount) {
            val executor = new Executor(id,driverPort.toInt, driverHost)
            executor.start()

          }

        }
      }).start()

    }
  }

}

```

### 5.3 信息

```scala
package com.atguigu.Scala_chapter01.scala_distributeCom
case class Message(m : String) {

}
```

### 5.4 执行器

```scala
class Executor(val id: Int, val driverPort: Int, val driverHost: String) {

  def start() {

    // 创建一个客户端
    val driverRef = new Socket(driverHost, driverPort)
    val driverRefIn = new ObjectInputStream(driverRef.getInputStream)
    val task: Task = driverRefIn.readObject().asInstanceOf[Task]
    task.num = id
    val i: Int = task.Com()
    driverRef.shutdownInput()

    // 创建一个输出流
    val executorOut = new ObjectOutputStream(driverRef.getOutputStream)
    executorOut.writeObject(Message(s"exerutor=${id}&result=${i}"))
    executorOut.flush()
    driverRef.close()

  }

}

```

### 5.5 任务

```scala
class Task {

  var num: Int = _
  var func: (Int) => Int = _

  def Com()= {
    func(num)
  }

}
```

