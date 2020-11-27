# Hive 几种数据导入方式

这篇文章主要是总结 Hive 的几种常见的数据导入方式，我总结为四类：

* 从本地文件系统中导入数据到 Hive 表
* 从 HDFS 上导入数据到 Hive 表
* 从别的表中查询出响应的数据并导入到 Hive 表中
* 在创建表的时候通过从别的表中查询出响应的记录并插入到所创建的表中。

我会对每一种数据的导入进行实际的操作，因为纯粹的文字让人看起来很枯燥，而且学起来也很抽象。好了，开始操作！

## 1.从本地文件系统中导入数据到 Hive 表

先在Hive里面创建好表，如下：

```sql
hive (mydata)> CREATE TABLE dyzcs
             > (id int, name string, age int, tel string)
             > ROW FORMAT DELIMITED
             > FIELDS TERMINATED BY '\t'
             > STORED AS TEXTFILE;
```

这个表很简单，只有四个字段，具体含义我就不解释了。本地文件系统里面有个 **/home/data/dyzcs.txt** 文件，内容如下：

```
[centos@s183 /home/centos/data]$cat dyzcs.txt 
1	dyzcs	25	13188888888888
2	test	30	13888888888888
3	zs	34	899314121
```

**dyzcs.txt** 文件中的数据列式使用 **\t** 分割的，可以通过下面的语句将这个文件里面的数据导入到 **dyzcs** 表里面，操作如下：

```shell
hive (mydata)> load data local inpath '/home/centos/data/dyzcs.txt' into table dyzcs;
Loading data to table mydata.dyzcs
OK
Time taken: 1.237 seconds
```

这样就将 **dyzcs.txt** 里面的内容导入到 dyzcs 表里面去了(关于这里面的执行过程我接下来会写一篇文章仔细介绍)，可以到 dyzcs 表的数据目录下查看，如下命令：

```shell
hive (mydata)> dfs -ls /user/hive/warehouse/mydata.db/dyzcs;
Found 1 items
-rwx-wx-wx   1 centos supergroup         69 2020-11-27 15:57 /user/hive/warehouse/mydata.db/dyzcs/dyzcs.txt
```

数据的确导入到 **dyzcs** 表里面去了。

## 2.HDFS 上导入数据到 Hive 表

从本地文件系统中将数据导入到 Hive 表的过程中，其实是先将数据临时复制到 HDFS 的一个目录下（典型的情况是复制到上传用户的HDFS home 目录下,比如 /home/centos/data/），然后再将数据从那个临时目录下移动（注意，**这里说的是移动，不是复制！**）到对应的Hive表的数据目录里面。既然如此，那么Hive肯定支持将数据直接从HDFS上的一个目录移动到相应Hive表的数据目录下，假设有下面这个文件 /user/centos/add.txt，具体的操作如下：

```shell
[centos@s183 /home/centos/data]$hdfs dfs -cat /user/centos/add.txt
5       dyz1    23      131212121212
6       dyz2    24      134535353535
7       dyz3    25      132453535353
8       dyz4    26      154243434355
```

上面是需要插入数据的内容，这个文件是存放在 HDFS 上 /user/centos/ 目录（和一中提到的不同，一中提到的文件是存放在本地文件系统上）里面，我们可以通过下面的命令将这个文件里面的内容导入到 Hive 表中，具体操作如下：

```shell
hive (mydata)> load data inpath '/user/centos/add.txt' into table dyzcs;
Loading data to table mydata.dyzcs
OK
Time taken: 1.119 seconds

hive (mydata)> select * from dyzcs;
OK
dyzcs.id	dyzcs.name	dyzcs.age	dyzcs.tel
5	dyz1	23	131212121212
6	dyz2	24	134535353535
7	dyz3	25	132453535353
8	dyz4	26	154243434355
1	dyzcs	25	13188888888888
2	test	30	13888888888888
3	zs	34	899314121
Time taken: 0.239 seconds, Fetched: 7 row(s)
```

Hive表在HDFS真实目录下文件结构如下

```shell
hive (mydata)> dfs -ls -R /user/hive/warehouse/mydata.db/dyzcs;
-rwx-wx-wx   1 centos supergroup         92 2020-11-27 16:15 /user/hive/warehouse/mydata.db/dyzcs/add.txt
-rwx-wx-wx   1 centos supergroup         69 2020-11-27 15:57 /user/hive/warehouse/mydata.db/dyzcs/dyzcs.txt
```

从上面的执行结果我们可以看到，数据的确导入到 dyzcs 表中了！请注意 load data inpath '/user/centos/add.txt' into table dyzcs; 里面是没有 **local** 这个单词的，这个是和一中的区别。

## 3.从别的表中查询出响应的数据并导入到Hive表中

假设 Hive 中有 test 表，其建表语句如下所示：

```sql
hive (mydata)> create table test
             > (id int, name string, tel string) 
             > partitioned by (age int)
             > row format delimited
             > fields terminated by '\t'
             > stored as textfile;
OK
Time taken: 0.192 seconds
```

大体和 dyzcs 表的建表语句类似，只不过 test 表里面用 age 作为了分区字段（关于什么是分区字段，其详细的介绍本博客将会在接下来的时间内介绍，请关注我）。下面语句就是将 dyzcs 表中的查询结果并插入到test表中：

```shell
hive (mydata)> insert into table test partition(age = '25') select id, name, tel from dyzcs;
#####################################################################
           这里输出了一堆Mapreduce任务信息，这里省略
#####################################################################
Loading data to table mydata.test partition (age=25)
OK
id	name	tel
Time taken: 12.68 seconds

hive (mydata)> select * from test;
OK
test.id	test.name	test.tel	test.age
5	dyz1	131212121212	25
6	dyz2	134535353535	25
7	dyz3	132453535353	25
8	dyz4	154243434355	25
1	dyzcs	13188888888888	25
2	test	13888888888888	25
3	zs	899314121	25
Time taken: 0.337 seconds, Fetched: 7 row(s)
```

通过上面的输出，我们可以看到从 dyzcs 表中查询出来的东西已经成功插入到 test 表中去了！如果目标表 (test) 中不存在分区字段，可以去掉 partition (age='25') 语句。当然，我们也可以在 select 语句里面通过使用分区值来动态指明分区：

```shell
hive (mydata)> set hive.exec.dynamic.partition.mode=nonstrict;
hive (mydata)> insert into table test
    > partition (age)
    > select id, name,
    > tel, age
    > from dyzcs;
#####################################################################
           这里输出了一堆Mapreduce任务信息，这里省略
#####################################################################
Time taken: 7.616 seconds
 
 
hive (mydata)> select * from test;
OK
5       wyp1    131212121212    23
6       wyp2    134535353535    24
7       wyp3    132453535353    25
1       wyp     13188888888888  25
8       wyp4    154243434355    26
2       test    13888888888888  30
3       zs      899314121       34
Time taken: 0.399 seconds, Fetched: 7 row(s)
```

这种方法叫做动态分区插入，但是 Hive 中默认是关闭的，所以在使用前需要先把 **hive.exec.dynamic.partition.mode** 设置为 **nonstrict** 。当然，Hive 也支持 insert overwrite 方式来插入数据，从字面我们就可以看出，overwrite 是覆盖的意思，是的，执行完这条语句的时候，相应数据目录下的数据将会被覆盖！而 insert into 则不会，注意两者之间的区别。例子如下：

```shell
insert overwrite table test PARTITION(age) select id, name, tel, age from dyzcs;
```

Hive还支持多表插入，什么意思呢？在 Hive 中，我们可以把 insert 语句倒过来，把 from 放在最前面，它的执行效果和放在后面是一样的，如下：

```shell
hive (mydata)> create table test2
             > (id int, name string)
             > row format delimited
             > fields terminated by '\t'
             > stored as textfile;

hive (mydata)> from dyzcs
             > insert overwrite table test partition(age)
             > select id, name, tel, age
             > insert into table test2
             > select id, name
             > where age > 24;

hive (mydata)> select * from test2;
OK
test2.id	test2.name
7	dyz3
8	dyz4
1	dyzcs
2	test
3	zs
Time taken: 0.201 seconds, Fetched: 5 row(s)
```

可以在同一个查询中使用多个 insert 子句，这样的好处是我们只需要扫描一遍源表就可以生成多个不相交的输出。这个很酷吧！

## 4.在创建表的时候通过从别的表中查询中相应的记录并插入到所创建的表中

在实际情况中，表的输出结果可能太多，不适于显示在控制台上，这时候，将Hive的查询输出结果直接存在一个新的表中是非常方便的，我们称这种情况为 **CTAS**（create table .. as select）如下：

```shell
hive (mydata)> create table test3 as
             > select id, name, tel
             > from dyzcs;

hive (mydata)> select * from test3;
OK
test3.id	test3.name	test3.tel
5	dyz1	131212121212
6	dyz2	134535353535
7	dyz3	132453535353
8	dyz4	154243434355
1	dyzcs	13188888888888
2	test	13888888888888
3	zs	899314121
Time taken: 0.19 seconds, Fetched: 7 row(s)
```

数据就插入到 test3 表中去了，**CTAS** 操作是原子的，因此如果 select 查询由于某种原因而失败，新表是不会创建的！
好了，今天就到这，下期见！！！