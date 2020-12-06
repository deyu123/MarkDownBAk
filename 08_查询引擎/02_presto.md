# Presto 

### 1. 概念

```sql
-- 解决商业数据仓库交互式分析处理速度问题
-- 响应时间小于1s 到几分钟的场景
```

### 2.Presto 架构

> ![image-20201115103457572](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115103457572.png)

```sql
Presto是一个运行在多台服务器上的分布式系统。完整安装包括一个Coordinator和多个Worker。由客户端提交查询，从Presto命令行CLI提交到Coordinator。Coordinator进行解析，分析并执行查询计划，然后分发处理队列到Worker。

-- 1）Coordinator
Coordinator服务器是用来解析语句，执行计划分析和管理Presto的Worker结点。Presto安装必须有一个Coordinator和多个Worker。如果用于开发环境和测试，则一个Presto实例可以同时担任这两个角色。
Coordinator跟踪每个Work的活动情况并协调查询语句的执行。Coordinator为每个查询建立模型，模型包含多个Stage，每个Stage再转为Task分发到不同的Worker上执行。
Coordinator与Worker、Client通信是通过REST API。

-- 2）Worker
Worker是负责执行任务和处理数据。Worker从Connector获取数据。Worker之间会交换中间数据。Coordinator是负责从Worker获取结果并返回最终结果给Client。
当Worker启动时，会广播自己去发现 Coordinator，并告知 Coordinator它是可用，随时可以接受Task。
Worker与Coordinator、Worker通信是通过REST API。

-- 3）数据源
贯穿全文，你会看到一些术语：Connector、Catelog、Schema和Table。这些是Presto特定的数据源
（1）Connector
Connector是适配器，用于Presto和数据源（如Hive、RDBMS）的连接。你可以认为类似JDBC那样，但却是Presto的SPI的实现，使用标准的API来与不同的数据源交互。 
Presto有几个内建Connector：JMX的Connector、System Connector（用于访问内建的System table）、Hive的Connector、TPCH（用于TPC-H基准数据）。还有很多第三方的Connector，所以Presto可以访问不同数据源的数据。 
每个Catalog都有一个特定的Connector。如果你使用catelog配置文件，你会发现每个文件都必须包含connector.name属性，用于指定catelog管理器（创建特定的Connector使用）。一个或多个catelog用同样的connector是访问同样的数据库。例如，你有两个Hive集群。你可以在一个Presto集群上配置两个catelog，两个catelog都是用Hive Connector，从而达到可以查询两个Hive集群。
（2）Catelog
一个Catelog包含Schema和Connector。例如，你配置JMX的catelog，通过JXM Connector访问JXM信息。当你执行一条SQL语句时，可以同时运行在多个catelog。
Presto处理table时，是通过表的完全限定（fully-qualified）名来找到catelog。例如，一个表的权限定名是hive.test_data.test，则test是表名，test_data是schema，hive是catelog。 
Catelog的定义文件是在Presto的配置目录中。
（3）Schema
Schema是用于组织table。把catelog好schema结合在一起来包含一组的表。当通过Presto访问hive或Mysq时，一个schema会同时转为hive和mysql的同等概念。
（4）Table
Table跟关系型的表定义一样，但数据和表的映射是交给Connector。
```

### 3. Presto 数据模型

> <img src="https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115103942533.png" alt="image-20201115103942533" style="zoom:67%;" />

```sql
1）Presto采取三层表结构：
Catalog：对应某一类数据源，例如Hive的数据，或MySql的数据
Schema：对应MySql中的数据库
Table：对应MySql中的表

2）Presto的存储单元包括：
Page：多行数据的集合，包含多个列的数据，内部仅提供逻辑行，实际以列式存储。
Block：一列数据，根据不同类型的数据，通常采取不同的编码方式，了解这些编码方式，有助于自己的存储系统对接presto。

3）不同类型的Block：
（1）Array类型Block，应用于固定宽度的类型，例如int，long，double。block由两部分组成：
boolean valueIsNull[]表示每一行是否有值。
T values[] 每一行的具体值。
（2）可变宽度的Block，应用于String类数据，由三部分信息组成
Slice：所有行的数据拼接起来的字符串。
int offsets[]：每一行数据的起始便宜位置。每一行的长度等于下一行的起始便宜减去当前行的起始便宜。
boolean valueIsNull[] 表示某一行是否有值。如果有某一行无值，那么这一行的便宜量等于上一行的偏移量。
（3）固定宽度的String类型的block，所有行的数据拼接成一长串Slice，每一行的长度固定。
（4）字典block：对于某些列，distinct值较少，适合使用字典保存。主要有两部分组成： 
字典，可以是任意一种类型的block(甚至可以嵌套一个字典block)，block中的每一行按照顺序排序编号。
int ids[]表示每一行数据对应的value在字典中的编号。在查找时，首先找到某一行的id，然后到字典中获取真实的值。
```

### 4. 优缺点

```sql
-- 优点：
1）Presto与Hive对比，都能够处理PB级别的海量数据分析，但Presto是基于内存运算，减少没必要的硬盘IO，所以更快。
2）能够连接多个数据源，跨数据源连表查，如从Hive查询大量网站访问记录，然后从Mysql中匹配出设备信息。
3）部署也比Hive简单，因为Hive是基于HDFS的，需要先部署HDFS。

-- 缺点：
1）虽然能够处理PB级别的海量数据分析，但不是代表Presto把PB级别都放在内存中计算的。而是根据场景，如count，avg等聚合运算，是边读数据边计算，再清内存，再读数据再计算，这种耗的内存并不高。但是连表查，就可能产生大量的临时数据，因此速度会变慢，反而Hive此时会更擅长。 
2）为了达到实时查询，可能会想到用它直连MySql来操作查询，这效率并不会提升，瓶颈依然在MySql，此时还引入网络瓶颈，所以会比原本直接操作数据库要慢。
```

### 5. 查询SQL优化

```sql
-- 1）只选择使用必要的字段 
由于采用列式存储，选择需要的字段可加快字段的读取、减少数据量。避免采用*读取所有字段。
[GOOD]: SELECT time,user,host FROM tbl
[BAD]:  SELECT * FROM tbl

-- 2）过滤条件必须加上分区字段 
对于有分区的表，where语句中优先使用分区字段进行过滤。acct_day是分区字段，visit_time是具体访问时间。
[GOOD]: SELECT time,user,host FROM tbl where acct_day=20171101
[BAD]:  SELECT * FROM tbl where visit_time=20171101

-- 3）Group By语句优化 
合理安排Group by语句中字段顺序对性能有一定提升。将Group By语句中字段按照每个字段distinct数据多少进行降序排列。
[GOOD]: SELECT GROUP BY uid, gender
[BAD]:  SELECT GROUP BY gender, uid

-- 4）Order by时使用Limit 
Order by需要扫描数据到单个worker节点进行排序，导致单个worker需要大量内存。如果是查询Top N或者Bottom N，使用limit可减少排序计算和内存压力。
[GOOD]: SELECT * FROM tbl ORDER BY time LIMIT 100
[BAD]:  SELECT * FROM tbl ORDER BY time

-- 5）使用近似聚合函数 
Presto有一些近似聚合函数，对于允许有少量误差的查询场景，使用这些函数对查询性能有大幅提升。比如使用approx_distinct() 函数比Count(distinct x)有大概2.3%的误差。
SELECT approx_distinct(user_id) FROM access

-- 6）用regexp_like代替多个like语句 
Presto查询优化器没有对多个like语句进行优化，使用regexp_like对性能有较大提升
[GOOD]
SELECT
  ...
FROM
  access
WHERE
  regexp_like(method, 'GET|POST|PUT|DELETE')

[BAD]
SELECT
  ...
FROM
  access
WHERE
  method LIKE '%GET%' OR
  method LIKE '%POST%' OR
  method LIKE '%PUT%' OR
  method LIKE '%DELETE%'
  
-- 7）使用Join语句时将大表放在左边 
Presto中join的默认算法是broadcast join，即将join左边的表分割到多个worker，然后将join右边的表数据整个复制一份发送到每个worker进行计算。如果右边的表数据量太大，则可能会报内存溢出错误。
[GOOD] SELECT ... FROM large_table l join small_table s on l.id = s.id
[BAD] SELECT ... FROM small_table s join large_table l on l.id = s.id

-- 8）使用Rank函数代替row_number函数来获取Top N 
在进行一些分组排序场景时，使用rank函数性能更好。
[GOOD]
SELECT checksum(rnk)
FROM (
  SELECT rank() OVER (PARTITION BY l_orderkey, l_partkey ORDER BY l_shipdate DESC) AS rnk
  FROM lineitem
) t
WHERE rnk = 1

[BAD]
SELECT checksum(rnk)
FROM (
  SELECT row_number() OVER (PARTITION BY l_orderkey, l_partkey ORDER BY l_shipdate DESC) AS rnk
  FROM lineitem
) t
WHERE rnk = 1
```

### 6. 注意事项

```sql
ORC和Parquet都支持列式存储，但是ORC对Presto支持更好（Parquet对Impala支持更好）
对于列式存储而言，存储文件为二进制的，对于经常增删字段的表，建议不要使用列式存储（修改文件元数据代价大）。对比数据仓库，dwd层建议不要使用ORC，而dm层则建议使用。
```



### 7. Presto 使用

```sql
多的时候，在Presto上对数据库跨库查询，例如Mysql数据库。这个时候Presto的做法是从MySQL数据库端拉取最基本的数据，然后再去做进一步的处理，例如统计等聚合操作。
举个例子：
SELECT count(id) FROM table_1 WHERE condition=1;

上面的SQL语句会分为3个步骤进行：
（1）Presto发起到Mysql数据库进行查询
SELECT id FROM table_1 WHERE condition=1;
（2）对结果进行count计算
（3）返回结果
所以说，对于Presto来说，其跨库查询的瓶颈是在数据拉取这个步骤。若要提高数据统计的速度，可考虑把Mysql中相关的数据表定期转移到HDFS中，并转存为高效的列式存储格式ORC。
所以定时归档是一个很好的选择，这里还要注意，在归档的时候我们要选择一个归档字段，如果是按日归档，我们可以用日期作为这个字段的值，采用yyyyMMdd的形式，例如20180123.
一般创建归档数据库的SQL语句如下：
CREATE TABLE IF NOT EXISTS table_1 (
id INTEGER,
........
partition_date INTEGER
)WITH ( format = 'ORC', partitioned_by = ARRAY['partition_date'] );
查看创建的库结构：
SHOW CREATE TABLE table_1; /*Only Presto*/
带有分区的表创建完成之后，每天只要更新分区字段partition_date就可以了，聪明的Presto就能将数据放置到规划好的分区了。
如果要查看一个数据表的分区字段是什么，可以下面的语句：
SHOW PARTITIONS FROM table_1 /*Only Presto*/


--  5.2 查询条件中尽量带上分区字段进行过滤
如果数据被规当到HDFS中，并带有分区字段。在每次查询归档表的时候，要带上分区字段作为过滤条件，这样可以加快查询速度。因为有了分区字段作为查询条件，就能帮助Presto避免全区扫描，减少Presto需要扫描的HDFS的文件数。


-- 5.3 多多使用WITH语句
使用Presto分析统计数据时，可考虑把多次查询合并为一次查询，用Presto提供的子查询完成。
这点和我们熟知的MySQL的使用不是很一样。
例如：
WITH subquery_1 AS (
    SELECT a1, a2, a3 
    FROM Table_1 
    WHERE a3 between 20180101 and 20180131
),               /*子查询subquery_1,注意：多个子查询需要用逗号分隔*/
subquery_2 AS (
    SELECT b1, b2, b3
    FROM Table_2
    WHERE b3 between 20180101 and 20180131
)                /*最后一个子查询后不要带逗号，不然会报错。*/        
SELECT 
    subquery_1.a1, subquery_1.a2, 
    subquery_2.b1, subquery_2.b2
FROM subquery_1
    JOIN subquery_2
    ON subquery_1.a3 = subquery_2.b3; 
-- 5.4 利用子查询，减少读表的次数，尤其是大数据量的表
具体做法是，将使用频繁的表作为一个子查询抽离出来，避免多次read。

-- 5.5 只查询需要的字段
一定要避免在查询中使用 SELECT *这样的语句，换位思考，如果让你去查询数据是不是告诉你的越具体，工作效率越高呢。
对于我们的数据库而言也是这样，任务越明确，工作效率越高。
对于要查询全部字段的需求也是这样，没有偷懒的捷径，把它们都写出来。

-- 5.6 Join查询优化
Join左边尽量放小数据量的表，而且最好是重复关联键少的表

-- 5.7 字段名引用
Presto中的字段名引用使用双引号分割，这个要区别于MySQL的反引号`。
当然，你可以不加这个双引号。

-- 5.8 时间函数
对于timestamp，需要进行比较的时候，需要添加timestamp关键字，而MySQL中对timestamp可以直接进行比较。
/*MySQL的写法*/
SELECT t FROM a WHERE t > '2017-01-01 00:00:00'; 

/*Presto中的写法*/
SELECT t FROM a WHERE t > timestamp '2017-01-01 00:00:00';

-- 5.9 MD5函数的使用
Presto中MD5函数传入的是binary类型，返回的也是binary类型，要对字符串进行MD5操作时，需要转换.
SELECT to_hex(md5(to_utf8('1212')));

-- 5.10 不支持INSERT OVERWRITE语法
Presto中不支持insert overwrite语法，只能先delete，然后insert into。

-- 5.11 ORC格式
Presto中对ORC文件格式进行了针对性优化，但在impala中目前不支持ORC格式的表，hive中支持ORC格式的表，所以想用列式存储的时候可以优先考虑ORC格式。

-- 5.12 PARQUET格式
Presto目前支持parquet格式，支持查询，但不支持insert。
```

### 8. Impala 和 Presto 的对比

```sql
-- 1. 都是在内存中运行的，相比Hive , 没有磁盘的IO 
-- 2. 分区多时，性能严重下降
-- 3. Impala 支持文本文件，不能读取自定义的二进制文件
-- 4. 新的记录添加HDFS 数据目录, Impala 表需要刷新
```

