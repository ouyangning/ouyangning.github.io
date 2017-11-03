---
title: MySQL中删除重复数据只保留一条
date: 2017-10-26 11:24:58
tags: 
- SQL
categories: 
- 数据挖掘
- 数据清洗
---

# MySQL中删除重复数据只保留一条

## 情境

数据库中有一张表`bone` ，有如下字段

```sql
lblCollectionUnicode, lblTitleName, lblRegisterType, lblType, lblFunctions, lblDimension, lblChineseDisplay, lblWestDisplay, lblDescription, imgShowImage
```

其中有几条数据因为插入了多次，所以是完全重复的，现在要求去除掉重复数据只保留一条。

### 思路

首先确定一个应该具有唯一性的字段，比如`lblCollectionUnicode` 。然后利用`SELECT` 语句找出所有`lblCollectionUnicode` 字段重复的元组，并依照其`lblCollectionUnicode` 字段的值进行分组。再利用`MIN()` 函数找出每个分组中`auto_increment` 字段（即自动增长字段，必须被定义为主键使用）值最小或最大的元组，将其保留，而将其他数据删去。

由于这张表中还没有`auto_increment`字段，因此首先要创建这个字段。

## 实现

```sql
CREATE TABLE new_bone LIKE bone;   
INSERT INTO new_bone SELECT * FROM bone;  
```

首先将表备份。

```sql
alter table bone add column _id int auto_increment not null, add primary key(_id);
```

在表`bone` 中新增一个`_id` 字段为整型、自动增长、非空，并将其设为主键。

```sql
DELETE FROM bone
WHERE lblCollectionUnicode IN ( SELECT * FROM(
        SELECT lblCollectionUnicode FROM bone 
        GROUP BY 
			lblCollectionUnicode 
        HAVING 
			count(lblCollectionUnicode) > 1 )as temp1)
AND _id NOT IN (SELECT * FROM( 
		SELECT MIN(_id) FROM bone 
        GROUP BY
			lblCollectionUnicode
		HAVING
			count(lblCollectionUnicode) > 1 )as temp2);
```

按照`lblCollectionUnicode` 的取值分组，其中元组数大于1的组有重复数据。利用`MIN()` 函数取得每组中`_id` 值最小的元组并将其排除在外，删除元组数大于1的组中的其余数据，即只保留一条数据。

```sql
DELETE FROM bone
WHERE lblCollectionUnicode IN (
        SELECT lblCollectionUnicode FROM bone 
        GROUP BY 
			lblCollectionUnicode 
        HAVING 
			count(lblCollectionUnicode) > 1 )
AND _id NOT IN (
		SELECT MIN(_id) FROM bone 
        GROUP BY
			lblCollectionUnicode
 		HAVING
 			count(lblCollectionUnicode) > 1);
```

 MYSQL执行上述语句报错：

 报错信息如下：

 　　`错误代码： 1093`
 　　`You can't specify target table 'bone' for update in FROM clause`

意思是不能在同一语句中更新select出的同一张表元组的属性值

解决方法：将select出的结果通过中间表再select一遍即可。

```sql
ALTER TABLE bone change _id _id INT(10);
ALTER TABLE bone DROP PRIMARY KEY;
ALTER TABLE bone ADD PRIMARY KEY(lblCollectionUnicode);
```

删除自增长后再删除主键，然后将`lblCollectionUnicode` 设置为主键。

