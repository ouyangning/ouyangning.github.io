---
title: 导入CSV文件到MySQL
date: 2017-11-29 19:37:44
tags: 
- MySQL
categories:
- 数据挖掘
- 数据集选取或构建
---



从网上下载了一个[公开数据集](https://www.capitalbikeshare.com/system-data)，想要用来做一点小实验。下载下载之后发现这是一个`.csv`格式的文件，里面大概有50多万条数据。那么下一步就是要把这些数据导入到数据库里。

### 方案一

用`python`的`pymysql`库来操作，先连接数据库，然后创建数据表，再构造出`sql`语句，把数据一条一条导进去。

```python
import csv
import pymysql
import time


if __name__ == '__main__':
    start = time.time()
    with open('2016-Q1-Trips-History-Data.csv', 'r') as csv_file:
        spam_reader = csv.reader(csv_file, delimiter=',')
        headers = next(spam_reader)
        db = pymysql.connect('localhost', 'username', 'password', 'tripshistorydata')
        cursor = db.cursor()
        cursor.execute('DROP TABLE IF EXISTS temp;')
        sql = """
            create table temp(
            Duration int not null,
            StartDate char(50) not null,
            EndDate char(50) not null,
            StartStationNumber int not null,
            StartStation char(100) not null,
            EndStationNumber int not null,
            EndStation char(100) not null,
            BikeNumber Char(50) not null,
            MemberType char(50) not null);
            """
        cursor.execute(sql)
        for row in spam_reader:
            param = (
                int(row[0]), row[1], row[2], int(row[3]), row[4], int(row[5]), row[6], row[7], row[8]
            )
            sql = """insert into temp(
            Duration, StartDate, EndDate, StartStationNumber, StartStation, EndStationNumber, EndStation, BikeNumber, MemberType
            ) values(%s, %s, %s, %s, %s, %s, %s, %s, %s);"""
            try:
                cursor.execute(sql, param)
                db.commit()
                # 记住这句话
            except Exception as e:
                print(e)
                db.rollback()
        
        db.close()
    stop = time.time()
    cost = stop - start
    print(cost)
```

但是吧，这么写实在太慢，睡了一个午觉还没导入完。那不行啊，那么有没有更快的办法呢？

### 方案二

答案当然是有的，为什么会那么慢，那是因为每一次`insert`的时候都会`commit`一次，这样当然会消耗很多时间。如果等所有数据都`insert`之后，只执行一次`commit`，当然就会快很多了。

然后嘞，还可以去`my.ini`里修改`innodb_flush_log_at_trx_commit`这么一个参数。

+ 当`innodb_flush_log_at_trx_commit=1`时，每次事务提交时MySQL都会把`log buffer`的数据写入`log file`，并且flush，该模式为系统默认。

  > 该模式是最安全的，但也是最慢的一种方式。在`mysqld `服务崩溃或者服务器主机`crash`的情况下，`binary log `只有可能丢失最多一个语句或者一个事务。

+ 当`innodb_flush_log_at_trx_commit=0`时，`log buffer`将每秒一次地写入`log file`中，并且`log file`的flush操作同时进行。该模式下在事务提交的时候，不会主动触发写入磁盘的操作。

  > 该模式速度最快，但不太安全，`mysqld`进程的崩溃会导致上一秒钟所有事务数据的丢失。

+ 当`innodb_flush_log_at_trx_commit=2`时，每次事务提交时MySQL都会把`log buffer`的数据写入`log file`，但是flush操作并不会同时进行。该模式下，MySQL会每秒执行一次 flush操作。

  > 该模式速度较快，也比`innodb_flush_log_at_trx_commit=0`安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。

综合起来，设置`innodb_flush_log_at_trx_commit=2`，然后重启MySQL服务器

看看这次需要多长的时间。

<font color=red>`264.5988073348999s`</font>

### 方案三

**MySQL有一个高效的导入方法，那就是`load data infile`。 **

语法如下

```sql
load data  [low_priority] [local] infile 'file_name txt' [replace | ignore]
into table tbl_name
[fields
[terminated by't']
[OPTIONALLY] enclosed by '']
[escaped by'\' ]]
[lines terminated by'n']
[ignore number lines]
[(col_name,   )]
```

`load data infile`语句从一个文本文件中以很高的速度读入一个表中。使用这个命令之前，`mysqld`进程（服务）必须已经在运行。为了安全原因，当读取位于服务器上的文本文件时，文件必须处于数据库目录或可被所有人读取。另外，为了对服务器上文件使用`load data infile`，在服务器主机上你必须有`file`的权限。

1. 如果你指定关键词`low_priority`，那么MySQL将会等到没有其他人读这个表的时候，才把插入数据。
2. 如果指定`local`关键词，则表明从客户主机读文件。如果没指定`local`，文件必须位于服务器上。
3. `replace`和`ignore`关键词控制对现有的唯一键记录的重复的处理。如果你指定`replace`，新行将代替有相同的唯一键值的现有行。如果你指定`ignore`，跳过有唯一键的现有行的重复行的输入。如果你不指定任何一个选项，当找到重复键时，出现一个错误，并且文本文件的余下部分被忽略。
4. 分隔符
   1.  `fields`关键字指定了文件记段的分割格式，如果用到这个关键字，MySQL剖析器希望看到至少有下面的一个选项：
      + `terminated by` 字段的分隔符
      + `enclosed by` 字段括起字符
      + `escaped by`  转义字符\

代码如下：

```sql
LOAD DATA LOCAL INFILE 'FileName' REPLACE 
INTO TABLE TableName
FIELDS 
TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY ''
LINES TERMINATED BY '\n'
IGNORE 1 lines;
```

<font color=red>`64.813s`</font>