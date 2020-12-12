# Shell

***

### 1. Shell 中的变量

```shell
-- 1. $n
	 功能描述：
	 1) n 为数字, $0 是脚本的名称， $1-$9 是代表 1-9 个参数
	 2) 10 个以上的参数，需要用大括号包含 ${10}
	 
-- 2. $#
	 功能描述：
	 1） 获取所有参数的个数，常用于循环

-- 3. $*, $@
	 功能描述：
   1） $* 代表所有参数, $* 把所有参数看成一个整体
   2） $@ 代表所有参数, $@ 把每个参数区分对待

-- 4. $? 
	 功能描述：
	 1）最后一次执行的命令返回的状态， 值 = 0 : 证明上一个命令正确执行，
	 														  值 != 0 : 具体是哪个数，由命令自己决定，则证明上一个命令执行不正确
	 2）
 
```

### 2.条件判断

```sql
-- 1．基本语法
[ condition ]（注意condition前后要有空格）
注意：条件非空即为true，[ atguigu ]返回true，[] 返回false。

-- 2. 常用判断条件
（1）两个整数之间比较

= 字符串比较
-lt 小于（less than）			-le 小于等于（less equal）
-eq 等于（equal）				-gt 大于（greater than）
-ge 大于等于（greater equal）	-ne 不等于（Not equal）

（2）按照文件权限进行判断

-r 有读的权限（read）			-w 有写的权限（write）
-x 有执行的权限（execute）

（3）按照文件类型进行判断

-f 文件存在并且是一个常规的文件（file）
-e 文件存在（existence）		-d 文件存在并是一个目录（directory）
```

### 3. If 判断

```sql
-- 1．基本语法
if [ 条件判断式 ];then 
  程序 
fi 
或者 
if [ 条件判断式 ] 
  then 
    程序 
elif [ 条件判断式 ]
	then
		程序
else
	程序
fi

	注意事项：
（1）[ 条件判断式 ]，中括号和条件判断式之间必须有空格
（2）if后要有空格
```

### 4. Case 语句

```sql
-- 1．基本语法
case $变量名 in 
  "值1"） 
    如果变量的值等于值1，则执行程序1 
    ;; 
  "值2"） 
    如果变量的值等于值2，则执行程序2 
    ;; 
  …省略其他分支… 
  *） 
    如果变量的值都不是以上的值，则执行此程序 
    ;; 
esac
注意事项：
1)case行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束。
2)双分号“;;”表示命令序列结束，相当于java中的break。
3)最后的“*）”表示默认模式，相当于java中的default。

-- 2．案例实操
（1）输入一个数字，如果是1，则输出banzhang，如果是2，则输出cls，如果是其它，输出renyao。
[atguigu@hadoop101 datas]$ touch case.sh
[atguigu@hadoop101 datas]$ vim case.sh

!/bin/bash

case $1 in
"1")
        echo "banzhang";;
"2")
        echo "cls";;
*)
        echo "renyao";;
esac

[atguigu@hadoop101 datas]$ chmod 777 case.sh
[atguigu@hadoop101 datas]$ ./case.sh 1
1
```

### 5.While 循环

```sql
-- 1．基本语法
while [ 条件判断式 ] 
  do 
    程序
  done
  
-- 2．案例实操
	（1）从1加到100
[atguigu@hadoop101 datas]$ touch while.sh
[atguigu@hadoop101 datas]$ vim while.sh

#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
        s=$[$s+$i]
        i=$[$i+1]
done

echo $s

[atguigu@hadoop101 datas]$ chmod 777 while.sh 
[atguigu@hadoop101 datas]$ ./while.sh 
5050
```

### 6. Read 读取控制台输入

```shell
1．基本语法
	read(选项)(参数)
	选项：
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒）。
参数
	变量：指定读取值的变量名
	
	
2．案例实操
	（1）提示7秒内，读取控制台输入的名称
[atguigu@hadoop101 datas]$ touch read.sh
[atguigu@hadoop101 datas]$ vim read.sh

#!/bin/bash

read -t 7 -p "Enter your name in 7 seconds " NAME
echo $NAME

[atguigu@hadoop101 datas]$ ./read.sh 
Enter your name in 7 seconds xiaoze
xiaoze	
	
```

### 7.函数

```sql
-- 1.系统函数
1．basename基本语法
basename [string / pathname] [suffix]  	（功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。
选项：
suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。
2．案例实操
（1）截取该/home/atguigu/banzhang.txt路径的文件名称
[atguigu@hadoop101 datas]$ basename /home/atguigu/banzhang.txt 
banzhang.txt
[atguigu@hadoop101 datas]$ basename /home/atguigu/banzhang.txt .txt
banzhang
3.	dirname基本语法
	dirname 文件绝对路径		（功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分））
4．案例实操
（1）获取banzhang.txt文件的路径
[atguigu@hadoop101 ~]$ dirname /home/atguigu/banzhang.txt 
/home/atguigu

-- 2. 自定义函数
1．基本语法
[ function ] funname[()]
{
	Action;
	[return int;]
}
funname
2．经验技巧
	（1）必须在调用函数地方之前，先声明函数，shell脚本是逐行运行。不会像其它语言一样先编译。
	（2）函数返回值，只能通过$?系统变量获得，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)
3．案例实操
	（1）计算两个输入参数的和
[atguigu@hadoop101 datas]$ touch fun.sh
[atguigu@hadoop101 datas]$ vim fun.sh

#!/bin/bash
function sum()
{
    s=0
    s=$[ $1 + $2 ]
    echo "$s"
}

read -p "Please input the number1: " n1;
read -p "Please input the number2: " n2;
sum $n1 $n2;

[atguigu@hadoop101 datas]$ chmod 777 fun.sh
[atguigu@hadoop101 datas]$ ./fun.sh 
Please input the number1: 2
Please input the number2: 5
7
```



### 8.Shell 工具

```shell
###  1 cut
cut的工作就是“剪”，具体的说就是在文件中负责剪切数据用的。cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段输出。
1.基本用法
cut [选项参数]  filename
说明：默认分隔符是制表符
2.选项参数说明
表1-55
选项参数	功能
-f	列号，提取第几列
-d	分隔符，按照指定分隔符分割列
-c	指定具体的字符
3.案例实操
（0）数据准备
[atguigu@hadoop101 datas]$ touch cut.txt
[atguigu@hadoop101 datas]$ vim cut.txt
dong shen
guan zhen
wo  wo
lai  lai
le  le
（1）切割cut.txt第一列   每一行按照 空格分隔后 取第几个
[atguigu@hadoop101 datas]$ cut -d " " -f 1 cut.txt 
dong
guan
wo
lai
le
（2）切割cut.txt第二、三列
[atguigu@hadoop101 datas]$ cut -d " " -f 2,3 cut.txt 
shen
zhen
 wo
 lai
 le
（3）在cut.txt文件中切割出guan
[atguigu@hadoop101 datas]$ cat cut.txt | grep "guan" | cut -d " " -f 1
guan
（4）选取系统PATH变量值，第2个“：”开始后的所有路径：
[atguigu@hadoop101 datas]$ echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/atguigu/bin

[atguigu@hadoop102 datas]$ echo $PATH | cut -d: -f 2-
/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/atguigu/bin
（5）切割ifconfig 后打印的IP地址
[atguigu@hadoop101 datas]$ ifconfig eth0 | grep "inet addr" | cut -d: -f 2 | cut -d" " -f1
192.168.1.102

######### 2 sed
sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。
1.基本用法
sed [选项参数]  ‘command’  filename
2.选项参数说明
表1-56
选项参数	功能
-e	直接在指令列模式上进行sed的动作编辑。
-i	直接编辑文件
3.命令功能描述
表1-57
命令	功能描述
a 	新增，a的后面可以接字串，在下一行出现
d	删除
s	查找并替换 
4.案例实操
（0）数据准备
[atguigu@hadoop102 datas]$ touch sed.txt
[atguigu@hadoop102 datas]$ vim sed.txt
dong shen
guan zhen
wo  wo
lai  lai

le  le
（1）将“mei nv”这个单词插入到sed.txt第二行下，打印。
[atguigu@hadoop102 datas]$ sed '2a mei nv' sed.txt 
dong shen
guan zhen
mei nv
wo  wo
lai  lai

le  le
[atguigu@hadoop102 datas]$ cat sed.txt 
dong shen
guan zhen
wo  wo
lai  lai

le  le
注意：文件并没有改变
（2）删除sed.txt文件所有包含wo的行
[atguigu@hadoop102 datas]$ sed '/wo/d' sed.txt
dong shen
guan zhen
lai  lai

le  le
（3）将sed.txt文件中wo替换为ni
[atguigu@hadoop102 datas]$ sed 's/wo/ni/g' sed.txt 
dong shen
guan zhen
ni  ni
lai  lai

le  le
注意：‘g’表示global，全部替换
（4）将sed.txt文件中的第二行删除并将wo替换为ni
[atguigu@hadoop102 datas]$ sed -e '2d' -e 's/wo/ni/g' sed.txt 
dong shen
ni  ni
lai  lai
le  le


##### 3 awk

一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。
1.基本用法
awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename
pattern：表示AWK在数据中查找的内容，就是匹配模式
action：在找到匹配内容时所执行的一系列命令
2.选项参数说明
表1-55
选项参数	功能
-F	指定输入文件折分隔符
-v	赋值一个用户定义变量
3.案例实操
（0）数据准备
[atguigu@hadoop102 datas]$ sudo cp /etc/passwd ./
（1）搜索passwd文件以root关键字开头的所有行，并输出该行的第7列。
[atguigu@hadoop102 datas]$ awk -F: '/^root/{print $7}' passwd 
/bin/bash
（2）搜索passwd文件以root关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割。
[atguigu@hadoop102 datas]$ awk -F: '/^root/{print $1","$7}' passwd 
root,/bin/bash
注意：只有匹配了pattern的行才会执行action
（3）只显示/etc/passwd的第一列和第七列，以逗号分割，且在所有行前面添加列名user，shell在最后一行添加"dahaige，/bin/zuishuai"。
[atguigu@hadoop102 datas]$ awk -F : 'BEGIN{print "user, shell"} {print $1","$7} END{print "dahaige,/bin/zuishuai"}' passwd
user, shell
root,/bin/bash
bin,/sbin/nologin
。。。
atguigu,/bin/bash
dahaige,/bin/zuishuai
注意：BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行。
（4）将passwd文件中的用户id增加数值1并输出
[atguigu@hadoop102 datas]$ awk -v i=1 -F: '{print $3+i}' passwd
1
2
3
4
4.awk的内置变量
表1-56
变量	说明
FILENAME	文件名
NR	已读的记录数
NF	浏览记录的域的个数（切割后，列的个数）
5.案例实操
（1）统计passwd文件名，每行的行号，每行的列数
[atguigu@hadoop102 datas]$ awk -F: '{print "filename:"  FILENAME ", linenumber:" NR  ",columns:" NF}' passwd 
filename:passwd, linenumber:1,columns:7
filename:passwd, linenumber:2,columns:7
filename:passwd, linenumber:3,columns:7
	  （2）切割IP
[atguigu@hadoop102 datas]$ ifconfig eth0 | grep "inet addr" | awk -F: '{print $2}' | awk -F " " '{print $1}' 
192.168.1.102
	  （3）查询sed.txt中空行所在的行号
[atguigu@hadoop102 datas]$ awk '/^$/{print NR}' sed.txt 

###### 4 sort

sort命令是在Linux里非常有用，它将文件进行排序，并将排序结果标准输出。
1.基本语法
sort(选项)(参数)
表1-57
选项	说明
-n	依照数值的大小排序
-r	以相反的顺序来排序
-t	设置排序时所用的分隔字符
-k	指定需要排序的列
参数：指定待排序的文件列表
2. 案例实操
（0）数据准备
[atguigu@hadoop102 datas]$ touch sort.sh
[atguigu@hadoop102 datas]$ vim sort.sh 
bb:40:5.4
bd:20:4.2
xz:50:2.3
cls:10:3.5
ss:30:1.6
（1）按照“：”分割后的第三列倒序排序。
[atguigu@hadoop102 datas]$ sort -t : -nrk 3  sort.sh 
bb:40:5.4
bd:20:4.2
cls:10:3.5
xz:50:2.3
ss:30:1.6
```

### 9. 面试题

```sql
--  11.1 京东
问题1：使用Linux命令查询file1中空行所在的行号
答案：
[atguigu@hadoop102 datas]$ awk '/^$/{print NR}' sed.txt 
5
问题2：有文件chengji.txt内容如下:
张三 40
李四 50
王五 60
使用Linux命令计算第二列的和并输出
[atguigu@hadoop102 datas]$ cat chengji.txt | awk -F " " '{sum+=$2} END{print sum}'
150


--- 11.2 搜狐&和讯网
问题1：Shell脚本里如何检查一个文件是否存在？如果不存在该如何处理？
#!/bin/bash

if [ -f file.txt ]; then
   echo "文件存在!"
else
   echo "文件不存在!"
fi


-- 11.3 新浪
问题1：用shell写一个脚本，对文本中无序的一列数字排序
[root@CentOS6-2 ~]# cat test.txt
9
8
7
6
5
4
3
2
10
1
[root@CentOS6-2 ~]# sort -n test.txt|awk '{a+=$0;print $0}END{print "SUM="a}'
1
2
3
4
5
6
7
8
9
10
SUM=55
-- 11.3 金和网络
问题1：请用shell脚本写出查找当前文件夹（/home）下所有的文本文件内容中包含有字符”shen”的文件名称
[atguigu@hadoop102 datas]$ grep -r "shen" /home | cut -d ":" -f 1
/home/atguigu/datas/sed.txt
/home/atguigu/datas/cut.txt
```

***

## 常用命令:df、ps、top、iotop、netstat

| top                           | 查看内存                               |
| ----------------------------- | -------------------------------------- |
| df -h                         | 查看磁盘存储情况                       |
| iotop                         | 查看磁盘IO读写(yum install iotop安装） |
| iotop -o                      | 直接查看比较高的磁盘读写程序           |
| netstat -tunlp \| grep 端口号 | 查看端口占用情况                       |
| uptime                        | 查看报告系统运行时长及平均负载         |
| ps  aux                       | 查看进程                               |

## Shell 常用工具 （awk, sort, sed,cut)

| awk    | awk  -F              | 指定文件拆分符                    |
| ------ | -------------------- | --------------------------------- |
| awk -v | 赋值一个用户定义变量 |                                   |
| sort   | -n                   | 依照数值大小排序                  |
| -r     | 以相反的顺序排序     |                                   |
| -t     | 定排序时所用的栏位   |                                   |
| -k     | 指定需要排序的栏位   |                                   |
| sed    | -e                   | 直接在指令列模式上进行sed动作编辑 |
| -d     | 删除                 |                                   |
| -s     | 查找并替换           |                                   |
| cut    | -f                   | 取第几列                          |
| -d     | 指定分割符分割列     |                                   |

```shell
-- 判断文件是否存在
if [ -f file.txt] ; then
  echo "存在"
else 
  echo "不存在"
fi

-- 当前文件夹("/home") 所有的文本文件中包含 “shen” 的文件名称
grep -r "Hbase" ./  | cut -d ":" -f 1 


.//09_Hbase/HBase.md://创建HbaseAdmin实例
.//09_Hbase/HBase.md:HBaseAdmin hAdmin = new HBaseAdmin(HbaseConfiguration.create());
.//09_Hbase/HBase.md://通过HTableDescriptor实例和散列值二维数组创建带有预分区的Hbase表
.//09_Hbase/HBase.md:4. 通过insert命令将中间表中的数据导入到Hive关联Hbase的那张表中
.//09_Hbase/HBase.md:Hbase> scan 'hbase_emp_table'
.//09_Hbase/HBase.md:      -- 1). Hbase读取Phoenix表

-------------------------------

inary file .//.git/index matches
.//06_Kafka总结/02_Kafka 常见问题.md:1. 利用zk 的临时节点来实现分布式锁，谁先抢，谁做controlle, Hbase HMaster , canal 高可用都是使用分布式锁
deyu-deMacBook-Pro:MarkDown deyu$ grep -r "Hbase" ./  | cut -d ":" -f 1
.//08_查询引擎/01_impala.md
.//14_面试相关/面试.md
.//14_面试相关/面试.md
.//14_面试相关/面试准备.md
.//14_面试相关/面试准备.md

-- shell 写一个脚本，对文本无序文件进行排序
[root@CentOS6-2 ~]# cat test.txt
9
8
7
6
5
4
3
2
10
1

[root@CentOS6-2 ~]# sort -n test.txt| awk '${a+=$0;print $0} END {'sum='$0}'
1
2
3
4
5
6
7
8
9
10
SUM=55

```

