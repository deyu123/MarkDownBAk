# ElasticSearch

### 1. 与其他数据存储的对比

|               | redis        | mysql           | elasticsearch                                                | hbase                                                 | hadoop/hive     |
| ------------- | ------------ | --------------- | ------------------------------------------------------------ | ----------------------------------------------------- | --------------- |
| 容量/容量扩展 | 低           | 中              | 较大                                                         | 海量                                                  | 海量            |
| 查询时效性    | 极高         | 中等            | 较高                                                         | 中等                                                  | 低              |
| 查询灵活性    | 较差 k-v模式 | 非常好，支持sql | 较好，关联查询较弱，但是可以全文检索，DSL语言可以处理过滤、匹配、排序、聚合等各种操作 | 较差，主要靠rowkey,scan的话性能不行，或者建立二级索引 | 非常好，支持sql |
| 写入速度      | 极快         | 中等            | 较快                                                         | 较快                                                  | 慢              |
| 一致性、事务  | 弱           | 强              | 弱                                                           | 弱                                                    | 弱              |

### 2. elasticsearch的特点

#### 2.1 天然分片，天然集群

> es 把数据分成多个shard，下图中的P0-P2，多个shard可以组成一份完整的数据，这些shard可以分布在集群中的各个机器节点中。随着数据的不断增加，集群可以增加多个分片，把多个分片放到多个机子上，已达到负载均衡，横向扩展。
>
> ![image-20201115113341855](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115113341855.png) 
>
> 在实际运算过程中，每个查询任务提交到某一个节点，该节点必须负责将数据进行整理汇聚，再返回给客户端，也就是一个简单的节点上进行Map计算，在一个固定的节点上进行Reduces得到最终结果向客户端返回。
>
> ![image-20201115113357012](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115113357012.png)
>
>   **这种集群分片的机制造就了elasticsearch强大的数据容量及运算扩展性 **。

  

#### 2.2 天然索引

>ES 所有数据都是默认进行索引的，这点和mysql正好相反，mysql是默认不加索引，要加索引必须特别说明，ES只有不加索引才需要说明。而ES使用的是倒排索引和Mysql的B+Tree索引不同。
>
> 
>
>传统关系性数据库
>
>![image-20201115113559558](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115113559558.png)
>
>弊端：  
>
>1、 对于传统的关系性数据库对于关键词的查询，只能逐字逐行的匹配，性能非常差。
>
> 2、匹配方式不合理，比如搜索“小密手机” ，如果用like进行匹配， 根本匹配不到。但是考虑使用者的用户体验的话，除了完全匹配的记录，还应该显示一部分近似匹配的记录，至少应该匹配到“手机”。

​    

#### 2.3 倒排索引是怎么处理的

>  全文搜索引擎目前主流的索引技术就是倒排索引的方式。
>
> ![image-20201115114157440](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115114157440.png)
>
>  传统的保存数据的方式都是		记录→单词
>
> 而倒排索引的保存数据的方式是 	单词→记录
>
> 例如	搜索“红海行动”
>
> 但是数据库中保存的数据如图：
>
> 
>
> 那么搜索引擎是如何能将两者匹配上的呢？
>
> 基于分词技术构建倒排索引：
>
> 首先每个记录保存数据时，都不会直接存入数据库。系统先会对数据进行分词，然后以倒排索引结构保存。如下：
>
>  ![image-20201115114355115](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115114355115.png)
>
> 然后等到用户搜索的时候，会把搜索的关键词也进行分词，会把“红海行动”分词分成：红海和行动两个词。
>
> 这样的话，先用红海进行匹配，得到id=1和id=2的记录编号，再用行动匹配可以迅速定位id为1,3的记录。
>
> 那么全文索引通常，还会根据匹配程度进行打分，显然1号记录能匹配的次数更多。所以显示的时候以评分进行排序的话，1号记录会排到最前面。而2、3号记录也可以匹配到。



#### 2.4 索引结构对比

> ##### **B+Tree**
>
> ![image-20201115114707742](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115114707742.png)
>
> 
>
> lucence 倒排索引结构
>
> ![image-20201115114743294](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201115114743294.png)
>
> 可以看到 lucene  为倒排索引(Term Dictionary)部分又增加一层Term Index结构，用于快速定位，而这Term Index是缓存在内存中的，但mysql的B+tree不在内存中，所以整体来看ES速度更快，但同时也更消耗资源（内存、磁盘）。

### 3. lucene与elasticsearch的关系

```
咱们之前讲的处理分词，构建倒排索引，等等，都是这个叫lucene的做的。那么能不能说这个lucene就是搜索引擎呢？

还不能。lucene只是一个提供全文搜索功能类库的核心工具包，而真正使用它还需要一个完善的服务框架搭建起来的应用。

好比lucene是类似于发动机，而搜索引擎软件（ES,Solr）就是汽车。

目前市面上流行的搜索引擎软件，主流的就两款，elasticsearch和solr,这两款都是基于lucene的搭建的，可以独立部署启动的搜索引擎服务软件。由于内核相同，所以两者除了服务器安装、部署、管理、集群以外，对于数据的操作，修改、添加、保存、查询等等都十分类似。就好像都是支持sql语言的两种数据库软件。只要学会其中一个另一个很容易上手。

从实际企业使用情况来看，elasticSearch的市场份额逐步在取代solr，国内百度、京东、新浪都是基于elasticSearch实现的搜索功能。国外就更多了 像维基百科、GitHub、Stack Overflow等等也都是基于ES的。
```

### 4. elasticsearch 基本概念

| cluster  | 整个elasticsearch 默认就是集群状态，整个集群是一份完整、互备的数据。 |
| -------- | ------------------------------------------------------------ |
| node     | 集群中的一个节点，一般只一个进程就是一个node                 |
| shard    | 分片，即使是一个节点中的数据也会通过hash算法，分成多个片存放，默认是5片。（7.0默认改为1片） |
| index    | 相当于rdbms的database(5.x), 对于用户来说是一个逻辑数据库，虽然物理上会被分多个shard存放，也可能存放在多个node中。  6.x 7.x index相当于table |
| type     | 类似于rdbms的table，但是与其说像table，其实更像面向对象中的class , 同一Json的格式的数据集合。（6.x只允许建一个，7.0被废弃，造成index实际相当于table级） |
| document | 类似于rdbms的 row、面向对象里的object                        |
| field    | 相当于字段、属性                                             |

### 5. ES 中数据结构

> public class  Movie {	 String id;   String name;   Double doubanScore;   List<Actor> actorList;} public class Actor{String id;String name;}
>
> 这两个对象如果放在关系型数据库保存，会被拆成2张表，但是elasticsearch是用一个json来表示一个document。
>
> 所以他保存到es中应该是：
>
> { “id”:”1”, “name”:”operation red sea”, “doubanScore”:”8.5”, “actorList”:[  {“id”:”1”,”name”:”zhangyi”},{“id”:”2”,”name”:”haiqing”},{“id”:”3”,”name”:”zhanghanyu”}]}



