# Scala总结之下篇1

------

## 接scala总结之中篇

## 八、 集合 (重点 )

### 8.1 简介

```sql
1. Scala集合分为：
	 Seq ： 序列
     Set ： 集合
     Map ： 映射

2. 所有的集合都扩展了自身的Iterable特质

3. scala提供了可变和不可变集合
	可变集合 :    scala.collection.mutable
	不可变集合：  scala.collection.immutable
	
4. 不可变并不是变量本身的值不可变，而是变量指向的那个内存地址不可变
```

### 8.2 Array

> Array是不可变集合，是在scala.collection.immutable包下

#### 8.2.1 声明数组

```scala
//方式1：new
val array : Array[Stirng] = new Array[String](5)

//方式2： 调用apply（）方法
/*
1. 形参中的数据为数组中元素，调用此方法时，至少传递一个参数，不能无参
2. 数组元素的类型采用数据类型推断的方式
*/
val array1  = Array(1,2,3,4)
```

#### 8.2.2 添加元素

```scala
//1.添加数据时，使用的是小括号，不是中括号
array(0) = "scala"
array(1) = "java"

//2.打印数组，通过new和apply方式创建的数组打印的地址值有差异, "["代表数组
println(array)  //[Ljava.lang.String;@19bb089b
println(array1)  //[I@4563e9ab
```

#### 8.2.3 遍历数组

```scala
//方式1 ： for循环
for ( i <- array){
    println(i)
}

//方式2： mkString(形参)，返回一个字符串，形参为数组中元素的分割符
val str = array.mkString(",")
println(str)
```

#### 8.2.4 数组操作

```scala
//通过如下方式操作数组，都会生成一个新的数组，原数组的地址和元素保持不变
// 1. +:  在数组中索引为0的位置添加一个数据,如果集合的方法采用冒号结尾，那么运算规则从右向左执行
    val array3: Array[Any] = array .+:(5)
    println(array3.mkString(",")) //5,scala5,scala1,scala2,scala3,scala4

// 2. :+  在数组末尾位置添加一个元素
	 val array2 = array :+ 5
	 println(array2.mkString(",")) // scala5,scala1,scala2,scala3,scala4,5

//3. ++  将一个数组添加在当前数组的末尾位置
  val array4: Array[Any] = array ++ array2
    print(array4.mkString(",")) //scala5,scala1,scala2,scala3,scala4,scala5,scala1,scala2,scala3,scala4,5
```

### 8.3 ArrayBuffer

> 可变数组，在scala.collection.mutable包下，当对数组进行操作时，数据可变，在一定数据数量的范围内，地址保持不变

#### 8.3.1 声明数组

```scala
// 方式1：new    
val buffer = new ArrayBuffer[String]()
// 方式2：apply
val buffer1: ArrayBuffer[Int] = ArrayBuffer(1,2,3,:4)
```

#### 8.3.2 添加元素

```scala
//添加元素
buffer.append("a")
```

#### 8.3.3 遍历数组

```scala
//方式1 ： for循环
  for ( i <- buffer1){
      println(i)
    }

//方式2： mkString
val str = buffer1.mkStirng(",")
println(str) //1,2,3,4

//方式3：打印数组名 
println(buffer1) //ArrayBuffer(1, 2, 3, 4)

```

#### 8.3.4 数组操作

```scala
//增 ：insert（角标位置，插入的元素1,插入的元素2,...）
  buffer1.insert(0,10) //在角标为0的索引位置增加元素10
  println(buffer1)  // ArrayBuffer(10, 1, 2, 3, 4)


  buffer1.insert(0,1,2,3,4,5,6)//在角标为0的索引位置开始增加元素，依次增加1,2,3,4,5,6
  println(buffer1) //ArrayBuffer(1, 2, 3, 4, 5, 6, 1, 2, 3, 4)

//删 remove
 	buffer1.remove(0) //删除索引位置为0的数据
    println(buffer1)  // ArrayBuffer( 2, 3, 4, 5, 6, 1, 2, 3, 4)

    buffer1.remove(0,2) //从索引为0的位置开始，删除两个数组的两个元素
    println(buffer1) // ArrayBuffer( 4, 5, 6, 1, 2, 3, 4)

//改 ：直接索引或者使用update
 	buffer1(0) = 10 //将索引位置为0的数据修改为10 ，编译时，自动转换为update()方法
    println(buffer1) // ArrayBuffer( 10, 5, 6, 1, 2, 3, 4)

   buffer1.update(1,20) //将索引位置为1的位置数据修改为20
    println(buffer1)// ArrayBuffer( 10,20, 6, 1, 2, 3, 4)

```

#### 8.3.5 Array和ArrayBuffer的区别

```sql
-- 可变数组
     内部存储的数据可以动态操作，而不会产生新的集合
     可变数组提供了大量对数据操作的方法，基本上方法名都是单词

-- 不可变数组
 	对数据的操作都会产生新的集合。
    提供对数据的操作方法相对来说，比较少，而且都是一些符号。
```

#### 8.3.6 Array和ArrayBuffer的转换

```scala
//  不可变  ==》 可变 ,toBuffer()
    val buffer2: mutable.Buffer[Int] = array.toBuffer

//   可变  ==》 不可变 ,toArray（）方法
   val array: Array[Int] = buffer1.toArray

```

### 8.4 声明集合

```scala
集合： 
  不可变        可变
1. Array  /  ArrayBuffer 
2. set    /   mutable.Set
3. List   /   ListBuffer 
4. Map    /   mutable.Map

/*统一采用apply()的方式:
1. 形参中的数据为集合中元素，调用此方法时，至少传递一个参数，不能无参
2. 集合元素的类型采用数据类型推断的方式
*/

//1-3 Array  /  ArrayBuffer 
    val array1  = Array(1,2,3,4)

//4  Map    /   mutable.Map
    //方式1：采用元组声明方式
    val map: Map[Int, String] = Map((1,"scala"),(2,"java"))
    //方式2：使用 "->" 方式
	val map1: Map[Int, String] = Map(1 -> "java" ,2 -> "scala")

```

### 8.5 遍历集合

```scala
  // 方式1 ： 直接打印，只有array不能使用这种方式
  // 方式2 ： for循环
  // 方式3 ： foreach
  // 方式4 ： 迭代器
  // 方式5 ： mkString()
```

```Scala
	val list = List(1,2,3,4)
    val map: Map[Int, String] = Map((1,"scala"),(2,"java"))
    val map1: Map[Int, String] = Map(1 -> "java" ,2 -> "scala")
   
	//方式1：直接打印，只有array不能使用这种方式
    println(list) //List(1, 2, 3, 4)

    // 方式2 ： for循环
    for ( i <- list ){
      println(i)
    }

    for(kv <- map){
      println(kv)
    }

    //方式3 ： foreach
    map.foreach(println)

    // 方式4 ： 迭代器
    val iterator: Iterator[(Int, String)] = map.iterator
    while (iterator.hasNext){
      println(iterator.next())
    }

    // 方式5 ：mkString()
    println(map.mkString(",")) //1 -> scala,2 -> java
```

### 8.6 集合默认情况

```sql
   关于集合默认情况介绍
    1.默认情况下scala提供的集合都是不可变的
    2.不可变集合List是一个抽象类，不能实例化，使用apply()方法进行创建对象
    3.set集合数据：无序、不可重复
    4.List集合数据：有序，可重复
    5.Map集合数据：KV键值对，k不可重复，无序
    6.我们在实际环境中，一般都是使用不可变集合。
    7.map中的一个键值对就是一个对偶元组【元组中的元素只有2个】
    8.函数式编程中，使用最多的算法就是递归算法
    9.scala中的元组自动比较大小，先比较第一个元素，再比较第二个元素，依次类推
    10.scala中的字符串默认排序方式是按照字典的顺序
    11.可迭代的集合之间都是可以互相转换的。
```

### 8.7 Nil 空集合

```sql
-- 1. 什么是空集合：集合中没有数据，也不能添加数据
	Nil = List()
-- 2. Nil作用：用于添加数据，会创建一个新的集合
-- 3. 添加数据的方式
```

```scala
	val list: List[Nothing] = List()
    val list1 = List("scala","java")

    //方式1
    val list2: List[Int] = 1 :: 2 :: 3 :: list
    println(list2) //List(1, 2, 3)

    //方式2:此时将list1作为一个整体添加到集合中
    val list3: List[Any] = 1 :: 2 :: list1 :: list
    println(list3) //List(1, 2, List(scala, java))

    //方式3：将list1集合中的数据拆分成一个一个的数据加入到集合中 【扁平化操作】
    val list4: List[Any] = 1 :: 2 :: list1 ::: list
    println(list4) //List(1, 2, scala, java)
```

### 8.8  Set （集合）

- mutable.Set的操作数据方法

```scala
    val set: mutable.Set[Int] = mutable.Set(1,2,3,3,4,6)
    println(set) // 重复的数据不能添加进去

    // 1. 添加数据 ： add（元素）
    set.add(10)
    println(set)

    // 2. 修改数据 ： undate(形参1，形参2)
    /*
    undate(形参1，形参2)
    形参1：向集合中更新的数据
    形参2：
        true（可理解为添加数据）  
        	     当集合中有形参1数据时，不作任何处理
                 当集合中没有形参1数据时，则将形参1添加到集合中
        flase （可理解为删除数据） 
        	    当集合中有形参1数据时，将集合中的这个数据删除
                当集合中没有形参1数据时，不做任何处理
     */
    println(set)
    set.update(5,true)
    println(set)
    set.update(2,false)
    println(set)

    // 3. 删除数据 ：remove（数据）
    set.remove(12)
    println(set)
```

```scala
  // 4.添加数据
    val set1 : mutable.Set[Int] = set + 2  //产生一个新的集合
    println(set)    //Set(1, 5, 6, 3, 10, 4)
    println(set1)   //Set(1, 5, 2, 6, 3, 10, 4)

    val set2: mutable.Set[Int] = set += 8 //不会产生一个新的集合
    println(set)    //Set(1, 5, 6, 3, 10, 4, 8)
    println(set2)   //Set(1, 5, 6, 3, 10, 4, 8)
    

    // 5.删除数据

     set - 2  //产生一个新的集合
     set -= 8 //不会产生一个新的集合
```

### 8.9 Map（映射）

- mutable.Map的操作方法

```scala
	 //创建map集合
    val map: mutable.Map[Int, String] = mutable.Map(1 -> "scala" , 2 -> "java")

    // 1. 增加数据 put(key ,value)
    map.put(3, "hadoop")
    println(map)

    // 2. 修改数据 put(key ,value)
    map.put(1,"java")
    println(map)

    // 3. 删除数据 remove(key)
    map.remove(1)
    println(map)
```

- 获取指定key的value

```sql
 	1. map.get(key)，返回值类型是Option
        Option ：选项类型
        如果根据key能获取到值，则返回some
        如果根据key获取不到值，则返回 None
        Option的作用：主要解决空指针问题，因为在实际生产环境中，我们一般取出值以后就直接拿来使用，如果数据为空，那么就能有空指针异常。

    2. map.get(key).get : 获取key的value返回值。
		如果此时Option的返回时None，此时报java.util.NoSuchElementException: None.get异常。

    3. 为了解决2中当获取不到key的value值时异常，设定默认值的方式
    	map.get(key).getOrElse(-1) : 表示当获取不到key对应的value时，给默认值-1

    4. 同时Map还提供一个方法map.getOrElse(key，default)，和3中方法作用是相同的
```

```Scala
val Value: Option[String] = map.get(1)

//    println(Value.get) //java.util.NoSuchElementException: None.get
    println(Value.getOrElse(-1)) //-1

    println(map.getOrElse(1, -1)) //-1
```

### 8.10 Tuple (元组)

- 元组介绍

```sql
		1，“张三”，90
    1. 问题：如果将无关的数据当成一个整体来使用，封装成json、Bean、集合都不是一个好的选择。
       方案：scala中提供了一种特殊的结构来进行封装上述的需求，这种特殊的结构采用括号的方式，称
        	这种方式为元组。
        	(1,"张三",90)

    2. 元组的数据一般是没有关系的，通过数据的顺序进行访问；

    3. 一个元组中最多只能有22条数据；

    4. 但是这个22个数据，不限制数据类型，如可以是集合；

    5. 当元组中的元素只有两个时，则称这个元组为对偶元组，也叫键值对。
```

- 元组的操作

```scala
    // 1. 元组的声明	
    val tuple: (Int, String, Int, String) = (1,"zhangsan",20,"lisi")

    // 2. 访问元组中的数据 ：元组对象._顺序号

    println(tuple._1)
    println(tuple._2)
    println(tuple._3)
    println(tuple._4)

    // 3. 使用迭代器的方式遍历元组中的数据:productIterator
    val iterator: Iterator[Any] = tuple.productIterator
    while (iterator.hasNext){
      println(iterator.next())
    }

    // 4. 通过索引的方式：productElement(index),获取指定索引位置的数据，索引值需在元组索引范围内
    println(tuple.productElement(1))  //zhangsan
    println(tuple.productElement(6))  //IndexOutOfBoundsException
```

### 8.11 常用方法

#### 8.11.1 简单常用方法

- 数据准备

```scala
 	val list = List("scala","java","java","trait","tuple")
    val set: Set[Int] = Set(1,2,3)
    val map: Map[Int, String] = Map(1 -> "scala",2 -> "java")
```

##### 8.11.1.1 集合的长度

```sql
  	1. size:表示集合的长度，适用于List、Set和Map
    2. length：表示集合的长度，仅适用于List，不适用于Set和Map
```

```scala
 	println(list.size)
    println(set.size)
    println(map.size)
    println(list.length)
```

##### 8.11.1.2 判断集合是否非空

```sql
	1. isEmpty
```

```scala
  	println(list.isEmpty)
    println(set.isEmpty)
    println(map.isEmpty)
```

##### 8.11.1.3 简单的运算

```sql
  	1. 求和: sum 适用于集合中的元素为值类型对象
  	2. 求最大 : max ,字符串默认按照字典的顺序进行排序
    3. 求最小 ： min
    4. 求乘积 : product
```

```scala
	//求和: sum 适用于集合中的元素为值类型对象
    println(set.sum)

    // 求最大 : max ,字符串默认按照字典的顺序进行排序
    println(list.max)

    //求最小 ： min
    println(list.min)

    //求乘积 : product
    println(set.product)
```

##### 8.11.1.4 获取集合中的元素

```sql
   1. head   ：返回第一个元素
   2. last   ：返回最后一个元素
   3. tail   ：返回除第一个元素之外的所有元素
   4. init   ：返回除最后一个元素之外的所有元素
   5. reverse : 集合反转
```

```scala
    // head   ：返回第一个元素
    println(map.head)

    // last   ：返回最后一个元素
    println(list.last)

    // tail   ：返回除第一个元素之外的所有元素
    println(list.tail)

    // init   ：返回除最后一个元素之外的所有元素
    println(list.init)

    // resver : 集合反转
    println(list.reverse)

```

##### 8.11.1.5  判断数据是否存在

```sql
 1. contains()
```

```scala
  println(list.contains("java"))
```

##### 8.11.1.6 数据去重

```sql
 方式 1. distinct
 方式 2. toSet : 将集合转换为Set集合，自动去重操作,但是会改变集合的类型，所以一般使用方式1
```

##### 8.11.1.7  拿取数据

```sql
 1.take(形参)
      形参："按照数据在集合中存储的顺序"，从左向右获取前几个数据
 
 2.takeRight(形参)
      形参："按照数据在集合中存储的顺序"，从左向右获取最后几个数据
```

```scala
 	// 1. 获取集合前面2个数据
    println(list)
    println(list.take(2))

    // 2.获取集合中最后3个数据
    println(list.takeRight(3))
```

##### 8.11.1.8  删除数据

```sql
 1.drop(形参)
       形参："按照数据在集合中存储的顺序"，从左向右删除前几个数据
 
 2. dropRight(形参)
       形参："按照数据在集合中存储的顺序"，从左向右删除最后几个数据
```

```scala
    // 返回删除前两个数据后的集合
    println(list.drop(2))

    // 返回删除最后2个数据后的集合
    println(list.dropRight(2))
```

#### 8.11.2 映射map

```sql
    1.映射转换 ：集合A --> 集合B，将集合中所有的元素按照指定的规则进行转换变成新的集合；
    2.方法 : map(形参)，返回一个集合
    3.形参 ：形参是一个函数，函数的形参为集合中的一个一个的元素，返回为按照规则转换以后的数据
```

```scala
//案例1
    val list = List(1, 2, 3, 4, 5)
    //将集合中的元素乘2倍以后返回
    val newList: List[Int] = list.map(_ * 2)
    println(newList) //List(2, 4, 6, 8, 10)

//案例2
    val list1 = List("scala","hadoop","java","tuple","trait")
    //将原集合中的数据，取每个数据的第一字母并返回
    val newList1: List[String] = list1.map(_.substring(0,1))
    println(newList1)//List(s, h, j, t, t)
```

#### 8.11.3 扁平化flatten

```scala
    1.扁平化：将整体拆分成一个一个的个体使用;
    2.方法：flatten，返回一个拆分后的集合;
    3.注意：
        a、使用一次flatten默认只能对外层进行操作，对内层的数据无法进行操作
        b、如果集合中的元素为字符串，那么在底层存储时默认为char[]数组，所以可以拆分为一个一个的字符
```

```scala
    // 案例1
    val list = List(List(1,2,3,4),List("scala","java","trait","tuple"))
    println(list.flatten) //List(1, 2, 3, 4, scala, java, trait, tuple)

    // 案例2,将一个字符串作为了一个char[]数组【集合】
    val list1 = List("scala","hadoop","java")
    println(list1.flatten) //List(s, c, a, l, a, h, a, d, o, o, p, j, a, v, a)
```

#### 8.11.4 扁平映射flatmap

```sql
    1. 扁平映射 = 扁平化（"主要作用"） + 映射 （次要作用）
    2. 扁平化的规则可以自定义。
    3. 方法：flatMap(形参) ，返回一个经过扁平化并执行了映射的集合；
    4. 形参：是一个函数，函数的形参为原集合中的一个一个的元素（此时原集合中的元素也是一个集合），
            返回值为一个可迭代的集合，将一个一个元素先进行扁平化
            
    5."重点：先映射后扁平化，对函数体中返回的集合进行扁平化"
```

![1590633762766](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/1590633762766.png)

![image-20200529095153618](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200529095153.png)

```scala
 //案例1
    val list = List(List(1, 2, 3, 6), List(2, 6, 5, 4))
    val newList: List[Int] = list.flatMap(dataList => {
      /*
      1.将list集中的第一个元素List(1, 2, 3, 6)进行映射，
      返回一个新的集合
      2. 然后将进行映射以后的集合进行扁平化 (1, 2, 3, 6)
       */
      dataList.map(_ * 2)
    })
    println(newList)

//案例2：
    val list1 = List("scala java" , "hello java" ,"hello scala")
    val newList2: List[String] = list1.flatMap(dataList => {
      /*
       此时传进来的datalist是一个字符串,经过映射，返回一个集合，此时的字符串被看做成了一个集合使用，
       返回("s scala" ,"c scala", "a scala".... )
       对如上的集合进行扁平化，"s scala" ,"c scala", "a scala"
                */
      dataList.map(str => {
        str + " scala"
      })
    })
    println(newList2) //List(s scala, c scala, a scala, l scala, a scala, ...


 //案例3：
    val list3 = List("scala java" ,"java scala")
    val newList3: List[String] = list3.flatMap(str => {
      //通过切片返回一个集合，"scala java" -> (scala ,java)
      str.split(" ")

    })
    println(newList3)//List(scala, java, java, scala)
```

#### 8.11.5 过滤fliter

```sql
    1. 过滤：按照指定的规则，对集合中所有元素进行过滤
    2. 方法：filter(形参)，返回经过过滤以后的集合
    3. 形参：是一个函数，函数的形参为：集合中的一个一个的元素，返回值为true或false；
            true：满足条件保留在集合中；
            false：不满足条件，该数据将被过滤。
```

```scala
//案例1   
	val list = List(1,2,3,4,5)
    val newList: List[Int] = list.filter(num => {
      //元素中的值为偶数留下，非偶数的数据过滤
      num % 2 == 0
    })
    println(newList) //List(2, 4)

//案例2
    val list2 = List("scala","hadoop","hive")
    val newList2: List[String] = list2.filter(str => {
      //过滤掉原集合中非“s”开头的字符串
      str.startsWith("s")
    })
    println(newList2) //List(scala)
```

#### 8.11.6 分组groupby

```sql
 1. 分组：按照指定的规则将数据进行分组
 2. 方法：groupBy(形参)：返回一个map集合，key为形参函数返回值，是一个分组的集合，value为原集合中经过函数体计算以后的结果满足key的所有元素
 3. 形参：是一个函数，函数的形参为：集合中的一个一个的元素，返回值为：函数体返回值为指定的key
```

```scala
//案例1
    val list = List("hello","java","scala","scala","java")
    val groupList: Map[String, List[String]] = list.groupBy(str => {
      //函数返回结果
      str
    })
    println(groupList)
    //Map(scala -> List(scala, scala), java -> List(java, java), hello -> List(hello))

//案例2
    val list2 = List(1,2,3,4,5,6)
    val groupList2: Map[Int, List[Int]] = list2.groupBy(num => {
      //返回值0或1，所以结果分成两个组
      num % 2
    })
    println(groupList2)//Map(1 -> List(1, 3, 5), 0 -> List(2, 4, 6))
```

#### 8.11.7 排序sortby

```sql
    1. 排序：将集合中的元素按照指定的字段规则进行排序，默认为升序
    2. 方法：sortBy(形参) ： 返回一个排好序的集合
    3. 形参：是一个函数，函数的形参为集合中一个一个的元素，返回值为排序的字段
    4. 可以使用柯里户化的方法，指定排序的方向，是升序还是倒序
    5. 当需要进行多层排序时，可以将需要排序的字段放进"元组"中，因为元组默认有排序的功能，而且是从第一个元素依次进行比较大小，
    此时如果定义排序的方式，使用元组排序的方法："Ordering.Tuple2(Ordering.String, Ordering.String.reverse)"
```

```scala
 //案例1
    val list = List(1,2,4,6,5)
    val sortList: List[Int] = list.sortBy(num => {
      //按照元素的值进行升序排序
      num
    })
    println(sortList)//List(1, 2, 4, 5, 6)

 //案例2
    val list2 = List("scala","hadoop","java","trait","tuple")
    val sortList1: List[String] = list2.sortBy(str => {
      //按照字符串的第三个字母进行排序
      str.substring(2, 3)
    })(Ordering.String.reverse)//降序
    println(sortList1) //List(java, tuple, hadoop, scala, trait)

 // 案例3
    val list3 = List("scala","json","java","trait","tuple")
    val sortList3: List[String] = list3.sortBy(str => {
      //先按照第一个字母升序排序，如果第一个字母相同，则比较第二个字母
      (str.substring(0, 1), str.substring(1, 2))
    })
    println(sortList3)//List(java, json, scala, trait, tuple)
  }
```

### 8.12 WordCount案例

- 需求1：

```
读取一个文件中的数据，计算每个字符串出现的次数，并打印出出现次数最多的前3信息
```

- 代码

```scala
object Scala_WordCount {
  def main(args: Array[String]): Unit = {
   
    //1. 读取按行文件中的数据并使用一个集合来接收
    val dataList: List[String] = Source.fromFile("input/test").getLines().toList

    //2. 使用扁平化操作，将一行数据转化为一个一个的字符串
    val flatList: List[String] = dataList.flatMap(data => {
      data.split(" ")
    })

    //3. 使用分组函数，将相同的字符串进行统计进一个map中
    val gruopList: Map[String, List[String]] = flatList.groupBy(str => {
      str
    })


    //4. 使用映射，将map集合中的每个元组的第二个元素转换为集合的长度
    val mapList: Map[String, Int] = gruopList.map(tuple => {
      (tuple._1, tuple._2.length)
    })

    //5. 根据每个字符串出现的顺序进行降序排序,如果字符相同，那么根据字符串的降序进行排序
    val toList: List[(String, Int)] = mapList.toList
    val sortList: List[(String, Int)] = toList.sortBy(tuple => {
      (tuple._2, tuple._1)
    })(Ordering.Tuple2(Ordering.Int.reverse, Ordering.String.reverse))


    //6. 取前3条数据
    println(sortList.take(3))

  }

}
```

- 需求2：

```
求单词出现的前三名
```

- 代码：

```scala
object Scala1_WordCokunt {
  def main(args: Array[String]): Unit = {

    val list = List("hello scala", "hello spark", "hive hadoop")
    //求单词出现的前三名
    val result = list
      .flatMap(_.split(" "))//List("hello", "scala", "hello", "spark", "hive, "hadoop")
      .groupBy(word => { word})
      .map(tuple => (tuple._1, tuple._2.length))
      .toList
      .sortBy { case (_, count) => count }(Ordering.Int.reverse)
      .take(3)
    println(result)
  }
}
```

- 需求3：

```scala
求如下集合单词出现次数最多的前三名，元组的第二个元素为第一个元素出现的次数
val list = List(
      ("hello", 4),
      ("hello spark", 3),
      ("hello spark scala", 2),
      ("hello spark scala hive", 1)
    )
```

- 方式1

```scala
object Scala2_WordCount {
  def main(args: Array[String]): Unit = {

    val list = List(
      ("hello", 4),
      ("hello spark", 3),
      ("hello spark scala", 2),
      ("hello spark scala hive", 1)
    )
    val result=
    list.flatMap(tuple => {val str = (tuple._1 + " ") * tuple._2;str.split(" ")})
        .groupBy(word => word)
        .map(tuple =>{(tuple._1,tuple._2.length)})
        .toList.sortBy(_._2)(Ordering.Int.reverse)
        .take(3)
    println(result)
  }
}
```

- 方法2:

```Scala
object Scala3_WordCount {
  def main(args: Array[String]): Unit = {

    val list = List(
      ("hello", 4),
      ("hello spark", 3),
      ("hello spark scala", 2),
      ("hello spark scala hive", 1)
    )

    val result=
    list.flatMap(tuple =>{
     val words: Array[String] = tuple._1.split(" ")
      words.map(word =>{
        (word,tuple._2)
      })
    })
      .groupBy(_._1)
      .mapValues(t => {
        t.map{case(_,num)=>num}.sum
      })
      .toList
      .sortBy(_._2)(Ordering.Int.reverse)
      .take(3)
    println(result)

  }

}

```

