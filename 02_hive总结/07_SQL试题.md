# SQL 考题

### 1. 蚂蚁森林

```sql
下表记录了用户每天的蚂蚁森林低碳生活领取的记录流水。
table_name：user_low_carbon  
user_id 	  data_dt 		 low_carbon
用户     日期      减少碳排放（g）

蚂蚁森林植物换购表，用于记录申领环保植物所需要减少的碳排放量table_name:  plant_carbon   
plant_id 	plant_name 	low_carbon
植物编号	 	植物名		换购植物所需要的碳
7.8.1.2 原始数据样例
user_low_carbon：
user_id 	date_dt	low_carbon
u_001		2017/1/1	10
u_001		2017/1/2	150
u_001		2017/1/2	110
u_001		2017/1/2	10
u_001		2017/1/4	50
u_001		2017/1/4	10
u_001		2017/1/6	45
u_001		2017/1/6	90
u_002		2017/1/1	10
u_002		2017/1/2	150
u_002		2017/1/2	70
u_002		2017/1/3	30
u_002		2017/1/3	80
u_002		2017/1/4	150
u_002		2017/1/5	101
u_002		2017/1/6	68
…

plant_carbon：
plant_id	plant_name	plant_carbon
p001		梭梭树			17
p002		沙柳			19
p003		樟子树			146
p004		胡杨			215
…

1.创建表
create table user_low_carbon(user_id String,data_dt String,low_carbon int) row format delimited fields terminated by '\t';

create table plant_carbon(plant_id string,plant_name string,low_carbon int) row format delimited fields terminated by '\t';

2.加载数据
load data local inpath "/opt/module/data/low_carbon.txt" into table user_low_carbon;

load data local inpath "/opt/module/data/plant_carbon.txt" into table plant_carbon;

3.设置本地模式
set hive.exec.mode.local.auto=true;
7.8.1.2 题目一
蚂蚁森林植物申领统计
问题：假设2017年1月1日开始记录低碳数据（user_low_carbon），假设2017年10月1日之前满足申领条件的用户都申领了一颗p004-胡杨，
剩余的能量全部用来领取“p002-沙柳” 。
统计在10月1日累计申领“p002-沙柳” 排名前10的用户信息；以及他比后一名多领了几颗沙柳。
得到的统计结果如下表样式：
user_id  plant_count less_count(比后一名多领了几颗沙柳)
u_101    1000         100
u_088    900          400
u_103    500          …
SQL流程
（1）先获取在10月1日前low_carbon总和最大的11个人
select user_id,sum(low_carbon) low_carbon_sum 
from user_low_carbon
where datediff(regexp_replace(data_dt, "/", "-"),regexp_replace('2017/10/1', "/", "-")) < 0
group by user_id
order by low_carbon_sum
desc limit 11;t1
（2）查询出"胡杨"所需的低碳量
select low_carbon from plant_carbon where plant_id='p004';t2
（3）查询出"沙柳"所需的低碳量
select low_carbon from plant_carbon where plant_id='p002';t3
（4）计算出在申领一颗"胡杨"后可申领的"沙柳"棵数
select user_id,round((t1.low_carbon_sum-t2.low_carbon)/t3.low_carbon) plant_count,
from t1,t2,t3;t4
（5）将每一行的下一个申领棵数放在当前行
select user_id,plant_count,lead(plant_count,1,0) over(sort by plant_count desc) as leadCount from t4;t5
（6）计算最终的差集（前十名比下一名多多少棵）
select user_id,(plant_count-leadCount) from t5 limit 10;
（7）最终Sql
SELECT
    user_id,
    plant_count,
    (plant_count - leadCount)
FROM
    (
        SELECT
            user_id,
            plant_count,
            lead (plant_count, 1, 0) over (sort BY plant_count DESC) AS leadCount
        FROM
            (
                SELECT
                    user_id,
                    round(
                        (
                            t1.low_carbon_sum - t2.low_carbon
                        ) / t3.low_carbon
                    ) plant_count
                FROM
                    (
                        SELECT
                            user_id,
                            sum(low_carbon) low_carbon_sum
                        FROM
                            user_low_carbon
                        WHERE
                            datediff(
                                regexp_replace (data_dt, "/", "-"),
                                regexp_replace ('2017/10/1', "/", "-")
                            ) < 0
                        GROUP BY
                            user_id
                        ORDER BY
                            low_carbon_sum DESC
                        LIMIT 11
                    ) t1,
                    (
                        SELECT
                            low_carbon
                        FROM
                            plant_carbon
                        WHERE
                            plant_id = 'p004'
                    ) t2,
                    (
                        SELECT
                            low_carbon
                        FROM
                            plant_carbon
                        WHERE
                            plant_id = 'p002'
                    ) t3
            ) t4
    ) t5
LIMIT 10;
（8）结果展示
+----------+--------------+-------+--+
| user_id  | plant_count  |  _c2  |
+----------+--------------+-------+--+
| u_007    | 66.0         | 2.0   |
| u_013    | 64.0         | 10.0  |
| u_008    | 54.0         | 7.0   |
| u_005    | 47.0         | 1.0   |
| u_010    | 46.0         | 2.0   |
| u_014    | 44.0         | 5.0   |
| u_011    | 39.0         | 1.0   |
| u_009    | 38.0         | 6.0   |
| u_006    | 32.0         | 9.0   |
| u_002    | 23.0         | 1.0   |
+----------+--------------+-------+--+



-- 蚂蚁森林低碳用户排名分析
问题：查询user_low_carbon表中每日流水记录，条件为：
用户在2017年，连续三天（或以上）的天数里，
每天减少碳排放（low_carbon）都超过100g的用户低碳流水。
需要查询返回满足以上条件的user_low_carbon表中的记录流水。
例如用户u_002符合条件的记录如下，因为2017/1/2~2017/1/5连续四天的碳排放量之和都大于等于100g：
seq（key） user_id data_dt  low_carbon
xxxxx10    u_002  2017/1/2  150
xxxxx11    u_002  2017/1/2  70
xxxxx12    u_002  2017/1/3  30
xxxxx13    u_002  2017/1/3  80
xxxxx14    u_002  2017/1/4  150
xxxxx14    u_002  2017/1/5  101
备注：统计方法不限于sql、procedure、python,java等
第一种解法：
SQL流程
（1）按照用户及时间聚合，计算每个人每天的低碳量（2017年）
select user_id,data_dt,sum(low_carbon) low_carbon_sum from user_low_carbon
where substring(data_dt,1,4)='2017'
group BY user_id,data_dt
having low_carbon_sum>100;t1
（2）将每一条数据的前后各两条数据的时间放置在一行，默认值为（1970/7/1）
select user_id,
       data_dt,
       lag(data_dt,2,"1970/7/1") over(partition by user_id) as lag2Date,
       lag(data_dt,1,"1970/7/1") over(partition by user_id) as lag1Date,
       lead(data_dt,1,"1970/7/1") over(partition by user_id) as lead1Date,
       lead(data_dt,2,"1970/7/1") over(partition by user_id) as lead2Date
from t1;t2
（3）计算每一天数据时间与前后两条数据之间的差值
select user_id,
       data_dt,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lag2Date, "/", "-")) lag2,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lag1Date, "/", "-")) lag1,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lead1Date, "/", "-")) lead1,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lead2Date, "/", "-")) lead2
from (select user_id,
       data_dt,
       lag(data_dt,2,"1970/7/1") over(partition by user_id) as lag2Date,
       lag(data_dt,1,"1970/7/1") over(partition by user_id) as lag1Date,
       lead(data_dt,1,"1970/7/1") over(partition by user_id) as lead1Date,
       lead(data_dt,2,"1970/7/1") over(partition by user_id) as lead2Date
from t2;t3
（4）取出最终需要的值，连续3天的（user_id,data_dt）
select user_id,data_dt
from t3
where (lag2=2 and lag1 =1) or (lag1 =1 and lead1 = -1) or(lead1=-1 and lead2 = -2);t4
（5）与原表Join得到最终需要的结果
select t5.user_id,t5.data_dt,t5.low_carbon
from user_low_carbon t5
join t4
where t4.user_id = t5.user_id and t4.data_dt = t5.data_dt;
（6）最终Sql
select t5.user_id,t5.data_dt,t5.low_carbon
from user_low_carbon t5
join (select user_id,data_dt
from (select user_id,
       data_dt,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lag2Date, "/", "-")) lag2,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lag1Date, "/", "-")) lag1,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lead1Date, "/", "-")) lead1,
       datediff(regexp_replace(data_dt, "/", "-"),regexp_replace(lead2Date, "/", "-")) lead2
from (select user_id,
       data_dt,
       lag(data_dt,2,"1970/7/1") over(partition by user_id) as lag2Date,
       lag(data_dt,1,"1970/7/1") over(partition by user_id) as lag1Date,
       lead(data_dt,1,"1970/7/1") over(partition by user_id) as lead1Date,
       lead(data_dt,2,"1970/7/1") over(partition by user_id) as lead2Date
from (select user_id,data_dt,sum(low_carbon) low_carbon_sum from user_low_carbon
where substring(data_dt,1,4)='2017'
group BY user_id,data_dt
having low_carbon_sum>100)t1)t2)t3
where (lag2=2 and lag1 =1) or (lag1 =1 and lead1 = -1) or(lead1=-1 and lead2 = -2))t4
where t4.user_id = t5.user_id and t4.data_dt = t5.data_dt;

SELECT
    t5.user_id,
    t5.data_dt,
    t5.low_carbon
FROM
    user_low_carbon t5
JOIN (
    SELECT
        user_id,
        data_dt
    FROM
        (
            SELECT
                user_id,
                data_dt,
                datediff(
                    regexp_replace (data_dt, "/", "-"),
                    regexp_replace (lag2Date, "/", "-")
                ) lag2,
                datediff(
                    regexp_replace (data_dt, "/", "-"),
                    regexp_replace (lag1Date, "/", "-")
                ) lag1,
                datediff(
                    regexp_replace (data_dt, "/", "-"),
                    regexp_replace (lead1Date, "/", "-")
                ) lead1,
                datediff(
                    regexp_replace (data_dt, "/", "-"),
                    regexp_replace (lead2Date, "/", "-")
                ) lead2
            FROM
                (
                    SELECT
                        user_id,
                        data_dt,
                        lag (data_dt, 2, "1970/7/1") over (PARTITION BY user_id) AS lag2Date,
                        lag (data_dt, 1, "1970/7/1") over (PARTITION BY user_id) AS lag1Date,
                        lead (data_dt, 1, "1970/7/1") over (PARTITION BY user_id) AS lead1Date,
                        lead (data_dt, 2, "1970/7/1") over (PARTITION BY user_id) AS lead2Date
                    FROM
                        (
                            SELECT
                                user_id,
                                data_dt,
                                sum(low_carbon) low_carbon_sum
                            FROM
                                user_low_carbon
                            WHERE
                                substring(data_dt, 1, 4) = '2017'
                            GROUP BY
                                user_id,
                                data_dt
                            HAVING
                                low_carbon_sum > 100
                        ) t1
                ) t2
        ) t3
    WHERE
        (lag2 = 2 AND lag1 = 1)
    OR (lag1 = 1 AND lead1 = - 1)
    OR (lead1 =- 1 AND lead2 = - 2)
) t4
WHERE
    t4.user_id = t5.user_id
AND t4.data_dt = t5.data_dt;
（7）结果展示
+-------------+-------------+----------------+--+
| t5.user_id  | t5.data_dt  | t5.low_carbon  |
+-------------+-------------+----------------+--+
| u_002       | 2017/1/2    | 150            |
| u_002       | 2017/1/2    | 70             |
| u_002       | 2017/1/3    | 30             |
| u_002       | 2017/1/3    | 80             |
| u_002       | 2017/1/4    | 150            |
| u_002       | 2017/1/5    | 101            |
| u_005       | 2017/1/2    | 50             |
| u_005       | 2017/1/2    | 80             |
| u_005       | 2017/1/3    | 180            |
| u_005       | 2017/1/4    | 180            |
| u_005       | 2017/1/4    | 10             |
| u_008       | 2017/1/4    | 260            |
| u_008       | 2017/1/5    | 360            |
| u_008       | 2017/1/6    | 160            |
| u_008       | 2017/1/7    | 60             |
| u_008       | 2017/1/7    | 60             |
| u_009       | 2017/1/2    | 70             |
| u_009       | 2017/1/2    | 70             |
| u_009       | 2017/1/3    | 170            |
| u_009       | 2017/1/4    | 270            |
| u_010       | 2017/1/4    | 90             |
| u_010       | 2017/1/4    | 80             |
| u_010       | 2017/1/5    | 90             |
| u_010       | 2017/1/5    | 90             |
| u_010       | 2017/1/6    | 190            |
| u_010       | 2017/1/7    | 90             |
| u_010       | 2017/1/7    | 90             |
| u_011       | 2017/1/1    | 110            |
| u_011       | 2017/1/2    | 100            |
| u_011       | 2017/1/2    | 100            |
| u_011       | 2017/1/3    | 120            |
| u_013       | 2017/1/2    | 150            |
| u_013       | 2017/1/2    | 50             |
| u_013       | 2017/1/3    | 150            |
| u_013       | 2017/1/4    | 550            |
| u_013       | 2017/1/5    | 350            |
| u_014       | 2017/1/5    | 250            |
| u_014       | 2017/1/6    | 120            |
| u_014       | 2017/1/7    | 270            |
| u_014       | 2017/1/7    | 20             |
+-------------+-------------+----------------+--+
      
第二种解法：
      
SQL流程
（1）按照用户及时间聚合，计算每个人每天的低碳量（2017年）并给每一条数据打标签（同一个用户不同时间排序）
select user_id,data_dt,
sum(low_carbon) low_carbon_sum,
row_number() over(partition by user_id order by data_dt) as rn
from user_low_carbon
where substring(data_dt,1,4)='2017'
group BY user_id,data_dt
having low_carbon_sum>100;t1
      
（2）获取每一条数据时间跟标签之间的差值
select user_id,data_dt,date_sub(to_date(regexp_replace(data_dt,"/", "-")),rn) diffDate from t1;
      
（3）按照所获得的差值聚合，得到同一个用户下相同差值的个数
      
select user_id,data_dt,count(*) over(partition by user_id,diffDate) diffDateCount from t2;t3
      
（4）过滤出相同差值个数在3及以上的数据
select user_id,data_dt from t3 where diffDateCount>=3;
      
（5）与原表Join得到最终需要的结果
select t5.user_id,t5.data_dt,t5.low_carbon
from user_low_carbon t5
join t4
where t4.user_id = t5.user_id and t4.data_dt = t5.data_dt
order by t5.user_id,t5.data_dt;
      
（6）最终Sql
SELECT
    t5.user_id,
    t5.data_dt,
    t5.low_carbon
FROM
    user_low_carbon t5
JOIN (
    SELECT
        user_id,
        data_dt
    FROM
        (
            SELECT
                user_id,
                data_dt,
                count(*) over (
                    PARTITION BY user_id,
                    diffDate
                ) diffDateCount
            FROM
                (
                    SELECT
                        user_id,
                        data_dt,
                        date_sub(
                             to_date (
                                regexp_replace (data_dt, "/", "-")
                            ),
                            rn
                        ) diffDate
                    FROM
                        (
                            SELECT
                                user_id,
                                data_dt,
                                sum(low_carbon) low_carbon_sum,
                                row_number () over (
                                    PARTITION BY user_id
                                    ORDER BY
                                        data_dt
                                ) AS rn
                            FROM
                                user_low_carbon
                            WHERE
                                substring(data_dt, 1, 4) = '2017'
                            GROUP BY
                                user_id,
                                data_dt
                            HAVING
                                low_carbon_sum > 100
                        ) t1
                ) t2
        ) t3
    WHERE
        diffDateCount >= 3
) t4
WHERE
    t4.user_id = t5.user_id
AND t4.data_dt = t5.data_dt
ORDER BY
    t5.user_id,
    t5.data_dt;
      
      
（7）结果展示
+-------------+-------------+----------------+--+
| t5.user_id  | t5.data_dt  | t5.low_carbon  |
+-------------+-------------+----------------+--+
| u_002       | 2017/1/2    | 150            |
| u_002       | 2017/1/2    | 70             |
| u_002       | 2017/1/3    | 30             |
| u_002       | 2017/1/3    | 80             |
| u_002       | 2017/1/4    | 150            |
| u_002       | 2017/1/5    | 101            |
| u_005       | 2017/1/2    | 50             |
| u_005       | 2017/1/2    | 80             |
| u_005       | 2017/1/3    | 180            |
| u_005       | 2017/1/4    | 180            |
| u_005       | 2017/1/4    | 10             |
| u_008       | 2017/1/4    | 260            |
| u_008       | 2017/1/5    | 360            |
| u_008       | 2017/1/6    | 160            |
| u_008       | 2017/1/7    | 60             |
| u_008       | 2017/1/7    | 60             |
| u_009       | 2017/1/2    | 70             |
| u_009       | 2017/1/2    | 70             |
| u_009       | 2017/1/3    | 170            |
| u_009       | 2017/1/4    | 270            |
| u_010       | 2017/1/4    | 90             |
| u_010       | 2017/1/4    | 80             |
| u_010       | 2017/1/5    | 90             |
| u_010       | 2017/1/5    | 90             |
| u_010       | 2017/1/6    | 190            |
| u_010       | 2017/1/7    | 90             |
| u_010       | 2017/1/7    | 90             |
| u_011       | 2017/1/1    | 110            |
| u_011       | 2017/1/2    | 100            |
| u_011       | 2017/1/2    | 100            |
| u_011       | 2017/1/3    | 120            |
| u_013       | 2017/1/2    | 150            |
| u_013       | 2017/1/2    | 50             |
| u_013       | 2017/1/3    | 150            |
| u_013       | 2017/1/4    | 550            |
| u_013       | 2017/1/5    | 350            |
| u_014       | 2017/1/5    | 250            |
| u_014       | 2017/1/6    | 120            |
| u_014       | 2017/1/7    | 270            |
| u_014       | 2017/1/7    | 20             |
+-------------+-------------+----------------+--+      

```

```
订单id    城市id  配送员id    日期date 
求： 按天算 每个城市的配送员量前十的配送员及订单量：

select * from (
select 
city_id, employee_id,
rank() over(partition by city_id, employee_id order by order count desc) as rank
from (
select count(distinct order_id) as order_count from 
t_table
group by city_id, employee_id, t_date))
where rank >= 10 
```



### 求出连续三天有销售记录的店铺

1） 原始数据 

A,2017-10-11,300

A,2017-10-12,200

A,2017-10-13,100

A,2017-10-15,100

A,2017-10-16,300

A,2017-10-17,150

A,2017-10-18,340

A,2017-10-19,360

B,2017-10-11,400

B,2017-10-12,200

B,2017-10-15,600

C,2017-10-11,350

C,2017-10-13,250

C,2017-10-14,300

C,2017-10-15,400

C,2017-10-16,200

D,2017-10-13,500

E,2017-10-14,600

E,2017-10-15,500

D,2017-10-14,600

2） 分析 : 给每个用户一个编号,用日期减去编号,如果是同一天,那么就是连续的

A,2017-10-11,300,1,2017-10-10

A,2017-10-12,200,2,2017-10-10

A,2017-10-13,100,3,2017-10-10

A,2017-10-15,100,4,2017-10-11

A,2017-10-16,300,5,2017-10-11

A,2017-10-17,150,6,2017-10-11

A,2017-10-18,340,7,2017-10-11

A,2017-10-19,360,8,2017-10-11

B,2017-10-11,400

B,2017-10-12,200

B,2017-10-15,600

C,2017-10-11,350

C,2017-10-13,250

C,2017-10-14,300

C,2017-10-15,400

C,2017-10-16,200

D,2017-10-13,500

E,2017-10-14,600

E,2017-10-15,500

D,2017-10-14,600

#### 1. 建表

create table t_jd(shopid string,dt string,sale int)
row format delimited fields terminated by ',';

![image-20201118141555254](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141555254.png)

load data local inpath '/root/sale.dat' into table t_jd;

![image-20201118141608151](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141608151.png)

#### 2. 打编号

select shopid,dt,sale,
row_number() over(partition by shopid order by dt) as rn 
from t_jd;

![image-20201118141645530](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141645530.png)

结果：

![image-20201118141701370](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141701370.png)

根据编号生成连续日期：

select shopid,dt,sale,rn,

date_sub(to_date(dt),rn) as flag

from

(select shopid,dt,sale,

row_number() over(partition by shopid order by dt) as rn 

from t_jd) tmp;

![image-20201118141745198](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141745198.png)

结果：

![image-20201118141758516](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141758516.png)

分组，求Count：

select shopid,count(1) as cnt 

from

(select shopid,dt,sale,rn,

date_sub(to_date(dt),rn) as flag

from

(select shopid,dt,sale,row_number() over(partition by shopid order by dt) as rn 

from t_jd) tmp) tmp2

group by shopid,flag;

结果：

![image-20201118141842504](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141842504.png)

筛选出连续天数大于等于3的：

select shopid from

(select shopid,count(1) as cnt 

from

(select shopid,dt,sale,rn,

date_sub(to_date(dt),rn) as flag

from

(select shopid,dt,sale,

row_number() over(partition by shopid order by dt) as rn 

from t_jd) tmp) tmp2

group by shopid,flag) tmp3

where tmp3.cnt>=3;



![image-20201118141920209](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141920209.png)

结果：

![image-20201118141927313](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141927313.png)

去重：

select distinct shopid from

(select shopid,count(1) as cnt 

from

(select shopid,dt,sale,rn,

date_sub(to_date(dt),rn) as flag

from

(select shopid,dt,sale,

row_number() over(partition by shopid order by dt) as rn 

from t_jd) tmp) tmp2

group by shopid,flag) tmp3

where tmp3.cnt>=3;

![image-20201118141948465](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141948465.png)

结果：

![image-20201118141957532](https://md-picgo.oss-cn-beijing.aliyuncs.com/oss-md-pic/image-20201118141957532.png)

***



event_id , 用户各性别，各手机类型，各年龄阶段（10 -20， 20-30， 30-40），在30天内用户事件次数的TopN





