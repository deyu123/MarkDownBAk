# Git

***

## 一、Git简介

```sql
-- 1. Git是什么？
1. 是一个分布式版本控制系统
2. 以"行"为单位进行存储，可以监控每行的变化
3. 几乎所有的软件的代码管理现在都在使用git.

-- 2. Git的作用：
   a、"版本还原"
   b、"代码备份"
   c、"分支管理"：秒级创建一个分支，分支进行代码编辑时，互不影响
   d、"冲突解决"：当不同分支对同一行代码进行变更时，前后提交，就会出现代码冲突，Git能快速高效解决冲突问题
   e、"历史追查"：可以看到代码修改的记录；
   f、"版本记录"：可以回到任何一个版本
   g、"权限管理"：针对不同的人开始不同的权限。
```

## 二、 Git基本配置

### 2.1 配置设置

```shell
1. 设置用户名和邮箱
     a、git config --global user.name lianzp           
     b、git config --global user.email liazp@atguigu.com  #在开发过程中，写实际的邮箱地址

2. 查询配置及查询配置所在的路径
      a、git config --list                   #查询配置信息
      b、 git config --list --show-origin    #查询配置文件所在的位置

3. 初始化本地库
      a、git init 

4. 取消换行符转换的warning提醒    #Windows下的换行符为\nf，而在linux下是\n
      a、git config core.autocrlf false
      
```

| 命令                               | 含义                         |
| ---------------------------------- | ---------------------------- |
| git config --list                  | 查看所有配置                 |
| git config --list --show-origin    | 查看所有配置以及所在文件位置 |
| git config --global user.name xxx  | 设置git用户名                |
| git config --global user.email xxx | 设置git邮箱                  |
| git init                           | 初始化本地库                 |
| git config core.autocrlf false     | 取消换行符转换的warning提醒  |

### 2.2 配置级别

```
有三个级别：
1. 系统默认，位于git安装路径下；
2. 用户配置：在c盘下；
3. 项目配置：在当前项目仓库的配置文件中。
```

<img src="https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200622200152.png" alt="image-20200622200151902" style="zoom:50%;" />

![image-20200622200432163](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200622200522.png)

### 2.3 Git三个概念

```sql
三个概念：
1.工作区(Working Directory):就是你电脑本地硬盘目录
2.本地库(Repository):工作区有个隐藏目录.git，它就是Git的本地版本库
3.暂存区(stage):一般存放在"git目录"下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）
```

![image-20200622201647983](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200622201648.png)

![image-20200622201723045](https://lian-zp.oss-cn-shenzhen.aliyuncs.com/pic GO/20200622201723.png)

## 三、Git基本命令

### 3.1 工作区、暂存区、本地仓库之间的转换

```sql
1. 创建一个文件
2. 查询本地库的状态
   "git status"
3. 上传过程：
   a、工作区 -> 暂存区 ：git add file
   b、暂存区 -> 本地仓库： git commit -m "备注信息"
4. a、工作区变更不想要：git restore file
   b、提交到暂存区，想打回给工作区： git restore --staged file 
   c、本地仓库的文件来覆盖工作区的文件： git checkout file
   
5. 说明：
```

### 3.2 忽略文件

```sql
--1. 使用场景
    a、当有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表种时。 
--2. 实现方式：
    a、创建".gitignore" 的文件，列出要忽略的文件的模式 --文件的名字不能错
    b、模式匹配规则
        # 忽略所有的.a 文件 
        *.a

        # 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件 
        !lib.a

        # 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO. 不递归的忽略
        /TODO

        # 忽略任何目录下名为 build 的文件夹 递归的忽略
        build/

        # 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt 
        doc/*.txt

        # 忽略 doc/ 目录及其所有子目录下的 .pdf 文件 
        doc/**/*.pdf
```

### 3.3 版本切换

```sql
--1. 查询当前版本
  每个版本的id是使用hash的算法，是全球唯一
  a、 "git log" ：以完整格式查看本地库状态(查看提交历史)
  b、 "git log" --pretty=oneline ： 上面a的美化版
    	e7a64baa2d768bca354c26b1aa68388a9be6f91a (HEAD -> master) 4th code
    	bc1576f56a214cf0826e79f71d904ceaff075c8c 3th code
    	9e25982ac31aa75873d53e7e63246003746919ba 2th code
    	5c9d7b2595e5a2b691f1a6aa1e9c4010dedba4f5 1th code

  c、 "git reflog" : 查看所有操作的历史记录
     e7a64ba (HEAD -> master) HEAD@{0}: commit: 4th code
     bc1576f HEAD@{1}: commit: 3th code
     9e25982 HEAD@{2}: commit: 2th code
     5c9d7b2 HEAD@{3}: commit (initial): 1th code

--2. 返回前一个版本

--3. 返回前两个版本

--4. 返回前多个版本

--5. 通用返回往前往后版本方式
  

   
```

